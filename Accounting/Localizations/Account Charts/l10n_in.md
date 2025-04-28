# Odoo Module: l10n_in

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Indian - Accounting',
    'version': '2.0',
    'description': """
Indian Accounting: Chart of Account.
====================================

Indian accounting chart and localization.

Odoo allows to manage Indian Accounting by providing Two Formats Of Chart of Accounts i.e Indian Chart Of Accounts - Standard and Indian Chart Of Accounts - Schedule VI.

Note: The Schedule VI has been revised by MCA and is applicable for all Balance Sheet made after
31st March, 2011. The Format has done away with earlier two options of format of Balance
Sheet, now only Vertical format has been permitted Which is Supported By Odoo.
  """,
    'category': 'Accounting/Localizations/Account Charts',
    'depends': [
        'account_tax_python', 'base_vat',
    ],
    'data': [
        'security/l10n_in_security.xml',
        'security/ir.model.access.csv',
        'data/account_data.xml',
        'data/l10n_in_chart_data.xml',
        'data/account.account.template.csv',
        'data/l10n_in_chart_post_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_fiscal_position_data.xml',
        'data/l10n_in.port.code.csv',
        'data/res_country_state_data.xml',
        'data/uom_data.xml',
        'views/account_invoice_views.xml',
        'views/account_journal_views.xml',
        'views/res_config_settings_views.xml',
        'views/product_template_view.xml',
        'views/port_code_views.xml',
        'views/report_invoice.xml',
        'views/res_country_state_view.xml',
        'views/res_partner_views.xml',
        'views/account_tax_views.xml',
        'views/uom_uom_views.xml',
        'views/report_template.xml',
        'data/account_chart_template_data.xml'
    ],
    'demo': [
        'demo/demo_company.xml',
        'demo/res_partner_demo.xml',
        'demo/product_demo.xml',
        'demo/account_payment_demo.xml',
        'demo/account_invoice_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","tag_ids/id","reconcile"
"p10031","Inventories","10031","account.data_account_type_current_assets","l10n_in.indian_chart_template_standard","","False"
"p10040","Debtors","10040","account.data_account_type_receivable","l10n_in.indian_chart_template_standard","","True"
"p10041","Debtors (PoS)","10041","account.data_account_type_receivable","l10n_in.indian_chart_template_standard","","True"
"p10051","SGST Receivable","10051","account.data_account_type_current_assets","l10n_in.indian_chart_template_standard","l10n_in.sgst_tag_account","False"
"p10052","CGST Receivable","10052","account.data_account_type_current_assets","l10n_in.indian_chart_template_standard","l10n_in.cgst_tag_account","False"
"p10053","IGST Receivable","10053","account.data_account_type_current_assets","l10n_in.indian_chart_template_standard","l10n_in.igst_tag_account","False"
"p10057","Reverse Charge Tax Receivable","10057","account.data_account_type_current_assets","l10n_in.indian_chart_template_standard","","False"
"p10054","TDS Receivable","10058","account.data_account_type_current_assets","l10n_in.indian_chart_template_standard","","False"
"p10061","Deposit Account","10061","account.data_account_type_current_assets","l10n_in.indian_chart_template_standard","","False"
"p10071","Prepaid Insurance","10071","account.data_account_type_current_assets","l10n_in.indian_chart_template_standard","","False"
"p1011","Buildings","1011","account.data_account_type_fixed_assets","l10n_in.indian_chart_template_standard","","False"
"p1012","Land","1012","account.data_account_type_fixed_assets","l10n_in.indian_chart_template_standard","","False"
"p1013","Equipments","1013","account.data_account_type_fixed_assets","l10n_in.indian_chart_template_standard","","False"
"p1014","Vehicle","1014","account.data_account_type_fixed_assets","l10n_in.indian_chart_template_standard","","False"
"p1015","Computer/Laptops (Assets)","1015","account.data_account_type_fixed_assets","l10n_in.indian_chart_template_standard","","False"
"p1016","Furniture","1016","account.data_account_type_fixed_assets","l10n_in.indian_chart_template_standard","","False"
"p1017","Air Conditionar","1017","account.data_account_type_fixed_assets","l10n_in.indian_chart_template_standard","","False"
"p1018","Misc Assets","1018","account.data_account_type_fixed_assets","l10n_in.indian_chart_template_standard","","False"
"p1111","Capital Account","1111","account.data_account_type_current_liabilities","l10n_in.indian_chart_template_standard","","False"
"p1112","Reserve And Surplus Account","1112","account.data_account_type_current_liabilities","l10n_in.indian_chart_template_standard","","False"
"p11211","Creditors","11211","account.data_account_type_payable","l10n_in.indian_chart_template_standard","","True"
"p11221","Bank OD Account","11221","account.data_account_type_current_liabilities","l10n_in.indian_chart_template_standard","","False"
"p11222","Secured Loan Account","11222","account.data_account_type_current_liabilities","l10n_in.indian_chart_template_standard","","False"
"p11223","Unsecured Loan Account","11223","account.data_account_type_current_liabilities","l10n_in.indian_chart_template_standard","","False"
"p11231","TDS Payable","11231","account.data_account_type_current_liabilities","l10n_in.indian_chart_template_standard","","False"
"p11232","SGST Payable","11232","account.data_account_type_current_liabilities","l10n_in.indian_chart_template_standard","l10n_in.sgst_tag_account","False"
"p11233","CGST Payable","11233","account.data_account_type_current_liabilities","l10n_in.indian_chart_template_standard","l10n_in.cgst_tag_account","False"
"p11234","IGST Payable","11234","account.data_account_type_current_liabilities","l10n_in.indian_chart_template_standard","l10n_in.igst_tag_account","False"
"p11241","Wages Payable","11241","account.data_account_type_current_liabilities","l10n_in.indian_chart_template_standard","","False"
"p11242","Interest Payable","11242","account.data_account_type_current_liabilities","l10n_in.indian_chart_template_standard","","False"
"p11243","Notes Payable","11243","account.data_account_type_current_liabilities","l10n_in.indian_chart_template_standard","","False"
"p20011","Local Sales","20011","account.data_account_type_revenue","l10n_in.indian_chart_template_standard","","False"
"p20012","Retail Sales","20012","account.data_account_type_revenue","l10n_in.indian_chart_template_standard","","False"
"p20013","Export Sales","20013","account.data_account_type_revenue","l10n_in.indian_chart_template_standard","","False"
"p20021","Local Services","20021","account.data_account_type_revenue","l10n_in.indian_chart_template_standard","","False"
"p20022","Export Services","20022","account.data_account_type_revenue","l10n_in.indian_chart_template_standard","","False"
"p2010","Interest Revenues","2010","account.data_account_type_revenue","l10n_in.indian_chart_template_standard","","False"
"p2011","Gain on Sale of Assets","2011","account.data_account_type_revenue","l10n_in.indian_chart_template_standard","","False"
"2012","Write off Income","2012","account.data_account_type_revenue","l10n_in.indian_chart_template_standard","","False"
"p2013","Foreign Exchange Profit","2013","account.data_account_type_other_income","l10n_in.indian_chart_template_standard","","False"
"p2100","Electricity Expense","2100","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2101","Salary Expense","2101","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2102","Office Rent","2102","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2103","House Keeping Expense","2103","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2104","Postage And Courier Expense","2104","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2105","Internet Expense","2105","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2106","Telephone Expense","2106","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2107","Purchase Expense","2107","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2108","Computer/Laptop Accessories","2108","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2109","News Paper And Magazine","2109","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2110","Business Promotion","2110","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2111","Entertainment Expense","2111","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2112","Professional Services","2112","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2113","Bank Charges","2113","account.data_account_type_liquidity","l10n_in.indian_chart_template_standard","","False"
"p2114","Diwali Bonus/Gift","2114","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2115","Parts Purchase","2115","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2116","Repairing Expense","2116","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2117","Foreign Exchange Loss","2117","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p21181","Sales Commission Expense","21181","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p21182","Stationary Expense","21182","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p21183","Travelling Expense","21183","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2121","Opening Stock","2121","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2122","Purchase Stock","2122","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2123","Closing Stock","2123","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2131","Loss on Sale of Assets","2131","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"
"p2132","Write Off Expense","2132","account.data_account_type_expenses","l10n_in.indian_chart_template_standard","","False"

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_in.indian_chart_template_standard')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Account Tax Group -->
        <record id="sgst_group" model="account.tax.group">
            <field name="name">SGST</field>
        </record>
        <record id="cgst_group" model="account.tax.group">
            <field name="name">CGST</field>
        </record>
        <record id="igst_group" model="account.tax.group">
            <field name="name">IGST</field>
        </record>
        <record id="cess_group" model="account.tax.group">
            <field name="name">Cess</field>
        </record>
        <record id="gst_group" model="account.tax.group">
            <field name="name">GST</field>
        </record>
        <record id="exempt_group" model="account.tax.group">
            <field name="name">Exempt</field>
        </record>
        <record id="nil_rated_group" model="account.tax.group">
            <field name="name">Nil Rated</field>
        </record>

    </data>
</odoo>

```

## File: data\account_fiscal_position_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Fiscal Position Templates -->
    <record model="account.fiscal.position.template" id="fiscal_position_in_inter_state">
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="name">Inter State</field>
    </record>

    <record model="account.fiscal.position.template" id="fiscal_position_in_export">
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="name">Export</field>
    </record>

    <!-- Fiscal Position Tax Templates -->
    <record id="account_fiscal_position_tax_in_sale_1_inter" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_inter_state"/>
        <field name="tax_src_id" ref="sgst_sale_1"/>
        <field name="tax_dest_id" ref="igst_sale_1"/>
    </record>
    <record id="account_fiscal_position_tax_in_sale_2_inter" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_inter_state"/>
        <field name="tax_src_id" ref="sgst_sale_2"/>
        <field name="tax_dest_id" ref="igst_sale_2"/>
    </record>
    <record id="account_fiscal_position_tax_in_sale_5_inter" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_inter_state"/>
        <field name="tax_src_id" ref="sgst_sale_5"/>
        <field name="tax_dest_id" ref="igst_sale_5"/>
    </record>
    <record id="account_fiscal_position_tax_in_sale_12_inter" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_inter_state"/>
        <field name="tax_src_id" ref="sgst_sale_12"/>
        <field name="tax_dest_id" ref="igst_sale_12"/>
    </record>
    <record id="account_fiscal_position_tax_in_sale_18_inter" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_inter_state"/>
        <field name="tax_src_id" ref="sgst_sale_18"/>
        <field name="tax_dest_id" ref="igst_sale_18"/>
    </record>
    <record id="account_fiscal_position_tax_in_sale_28_inter" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_inter_state"/>
        <field name="tax_src_id" ref="sgst_sale_28"/>
        <field name="tax_dest_id" ref="igst_sale_28"/>
    </record>

    <record id="account_fiscal_position_tax_in_purchase_1_inter" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_inter_state"/>
        <field name="tax_src_id" ref="sgst_purchase_1"/>
        <field name="tax_dest_id" ref="igst_purchase_1"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_2_inter" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_inter_state"/>
        <field name="tax_src_id" ref="sgst_purchase_2"/>
        <field name="tax_dest_id" ref="igst_purchase_2"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_5_inter" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_inter_state"/>
        <field name="tax_src_id" ref="sgst_purchase_5"/>
        <field name="tax_dest_id" ref="igst_purchase_5"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_12_inter" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_inter_state"/>
        <field name="tax_src_id" ref="sgst_purchase_12"/>
        <field name="tax_dest_id" ref="igst_purchase_12"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_18_inter" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_inter_state"/>
        <field name="tax_src_id" ref="sgst_purchase_18"/>
        <field name="tax_dest_id" ref="igst_purchase_18"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_28_inter" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_inter_state"/>
        <field name="tax_src_id" ref="sgst_purchase_28"/>
        <field name="tax_dest_id" ref="igst_purchase_28"/>
    </record>

    <record id="account_fiscal_position_tax_in_sale_1_exp" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_export"/>
        <field name="tax_src_id" ref="sgst_sale_1"/>
        <field name="tax_dest_id" ref="igst_sale_1"/>
    </record>
    <record id="account_fiscal_position_tax_in_sale_2_exp" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_export"/>
        <field name="tax_src_id" ref="sgst_sale_2"/>
        <field name="tax_dest_id" ref="igst_sale_2"/>
    </record>
    <record id="account_fiscal_position_tax_in_sale_5_exp" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_export"/>
        <field name="tax_src_id" ref="sgst_sale_5"/>
        <field name="tax_dest_id" ref="igst_sale_5"/>
    </record>
    <record id="account_fiscal_position_tax_in_sale_12_exp" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_export"/>
        <field name="tax_src_id" ref="sgst_sale_12"/>
        <field name="tax_dest_id" ref="igst_sale_12"/>
    </record>
    <record id="account_fiscal_position_tax_in_sale_18_exp" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_export"/>
        <field name="tax_src_id" ref="sgst_sale_18"/>
        <field name="tax_dest_id" ref="igst_sale_18"/>
    </record>
    <record id="account_fiscal_position_tax_in_sale_28_exp" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_export"/>
        <field name="tax_src_id" ref="sgst_sale_28"/>
        <field name="tax_dest_id" ref="igst_sale_28"/>
    </record>

    <record id="account_fiscal_position_tax_in_purchase_1_exp" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_export"/>
        <field name="tax_src_id" ref="sgst_purchase_1"/>
        <field name="tax_dest_id" ref="igst_purchase_1"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_2_exp" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_export"/>
        <field name="tax_src_id" ref="sgst_purchase_2"/>
        <field name="tax_dest_id" ref="igst_purchase_2"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_5_exp" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_export"/>
        <field name="tax_src_id" ref="sgst_purchase_5"/>
        <field name="tax_dest_id" ref="igst_purchase_5"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_12_exp" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_export"/>
        <field name="tax_src_id" ref="sgst_purchase_12"/>
        <field name="tax_dest_id" ref="igst_purchase_12"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_18_exp" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_export"/>
        <field name="tax_src_id" ref="sgst_purchase_18"/>
        <field name="tax_dest_id" ref="igst_purchase_18"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_28_exp" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_export"/>
        <field name="tax_src_id" ref="sgst_purchase_28"/>
        <field name="tax_dest_id" ref="igst_purchase_28"/>
    </record>

    <record model="account.fiscal.position.template" id="fiscal_position_in_reverse_charge_intra">
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="name">Reverse charge Intra State</field>
    </record>

    <record id="account_fiscal_position_tax_in_purchase_1_intra_rc" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_reverse_charge_intra"/>
        <field name="tax_src_id" ref="sgst_purchase_1"/>
        <field name="tax_dest_id" ref="sgst_purchase_1_rc"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_2_intra_rc" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_reverse_charge_intra"/>
        <field name="tax_src_id" ref="sgst_purchase_2"/>
        <field name="tax_dest_id" ref="sgst_purchase_2_rc"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_5_intra_rc" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_reverse_charge_intra"/>
        <field name="tax_src_id" ref="sgst_purchase_5"/>
        <field name="tax_dest_id" ref="sgst_purchase_5_rc"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_12_intra_rc" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_reverse_charge_intra"/>
        <field name="tax_src_id" ref="sgst_purchase_12"/>
        <field name="tax_dest_id" ref="sgst_purchase_12_rc"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_18_intra_rc" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_reverse_charge_intra"/>
        <field name="tax_src_id" ref="sgst_purchase_18"/>
        <field name="tax_dest_id" ref="sgst_purchase_18_rc"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_28_intra_rc" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_reverse_charge_intra"/>
        <field name="tax_src_id" ref="sgst_purchase_28"/>
        <field name="tax_dest_id" ref="sgst_purchase_28_rc"/>
    </record>

    <record model="account.fiscal.position.template" id="fiscal_position_in_reverse_charge_inter">
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="name">Reverse charge Inter State</field>
    </record>

    <record id="account_fiscal_position_tax_in_purchase_1_rc_inter_rc" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_reverse_charge_inter"/>
        <field name="tax_src_id" ref="sgst_purchase_1"/>
        <field name="tax_dest_id" ref="igst_purchase_1_rc"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_2_rc_inter_rc" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_reverse_charge_inter"/>
        <field name="tax_src_id" ref="sgst_purchase_2"/>
        <field name="tax_dest_id" ref="igst_purchase_2_rc"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_5_rc_inter_rc" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_reverse_charge_inter"/>
        <field name="tax_src_id" ref="sgst_purchase_5"/>
        <field name="tax_dest_id" ref="igst_purchase_5_rc"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_12_rc_inter_rc" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_reverse_charge_inter"/>
        <field name="tax_src_id" ref="sgst_purchase_12"/>
        <field name="tax_dest_id" ref="igst_purchase_12_rc"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_18_rc_inter_rc" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_reverse_charge_inter"/>
        <field name="tax_src_id" ref="sgst_purchase_18"/>
        <field name="tax_dest_id" ref="igst_purchase_18_rc"/>
    </record>
    <record id="account_fiscal_position_tax_in_purchase_28_rc_inter_rc" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_in_reverse_charge_inter"/>
        <field name="tax_src_id" ref="sgst_purchase_28"/>
        <field name="tax_dest_id" ref="igst_purchase_28_rc"/>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.in"/>
    </record>

    <record id="tax_report_line_gst" model="account.tax.report.line">
        <field name="name">GST</field>
        <field name="sequence">1</field>
        <field name="report_id" ref="tax_report"/>
    </record>
    <record id="tax_report_line_igst_root" model="account.tax.report.line">
        <field name="name">IGST</field>
        <field name="sequence">2</field>
        <field name="report_id" ref="tax_report"/>
    </record>
    <record id="tax_report_line_gst_others" model="account.tax.report.line">
        <field name="name">Others</field>
        <field name="sequence">3</field>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_sgst" model="account.tax.report.line">
        <field name="name">SGST</field>
        <field name="tag_name">SGST</field>
        <field name="parent_id" ref="tax_report_line_gst"/>
        <field name="sequence">1</field>
        <field name="report_id" ref="tax_report"/>
    </record>
    <record id="tax_report_line_cgst" model="account.tax.report.line">
        <field name="name">CGST</field>
        <field name="tag_name">CGST</field>
        <field name="parent_id" ref="tax_report_line_gst"/>
        <field name="sequence">2</field>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_igst" model="account.tax.report.line">
        <field name="name">IGST</field>
        <field name="tag_name">IGST</field>
        <field name="parent_id" ref="tax_report_line_igst_root"/>
        <field name="sequence">3</field>
        <field name="report_id" ref="tax_report"/>
    </record>
    <record id="tax_report_line_cess" model="account.tax.report.line">
        <field name="name">CESS</field>
        <field name="tag_name">CESS</field>
        <field name="parent_id" ref="tax_report_line_gst_others"/>
        <field name="sequence">4</field>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_sgst_rc" model="account.tax.report.line">
        <field name="name">SGST Reverse Charge</field>
        <field name="tag_name">SGST (RC)</field>
        <field name="parent_id" ref="tax_report_line_gst"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>
    <record id="tax_report_line_cgst_rc" model="account.tax.report.line">
        <field name="name">CGST Reverse Charge</field>
        <field name="tag_name">CGST (RC)</field>
        <field name="parent_id" ref="tax_report_line_gst"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
    </record>

    <record id="tax_report_line_igst_rc" model="account.tax.report.line">
        <field name="name">IGST Reverse Charge</field>
        <field name="tag_name">IGST (RC)</field>
        <field name="parent_id" ref="tax_report_line_igst_root"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
    </record>
    <record id="tax_report_line_cess_rc" model="account.tax.report.line">
        <field name="name">CESS Reverse Charge</field>
        <field name="tag_name">CESS (RC)</field>
        <field name="parent_id" ref="tax_report_line_gst_others"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">5</field>
    </record>

    <record id="tax_report_line_exempt" model="account.tax.report.line">
        <field name="name">Exempt</field>
        <field name="tag_name">Exempt</field>
        <field name="parent_id" ref="tax_report_line_gst_others"/>
        <field name="sequence">5</field>
        <field name="report_id" ref="tax_report"/>
    </record>
    <record id="tax_report_line_nil_rated" model="account.tax.report.line">
        <field name="name">Nil Rated</field>
        <field name="tag_name">Nil Rated</field>
        <field name="parent_id" ref="tax_report_line_gst_others"/>
        <field name="sequence">6</field>
        <field name="report_id" ref="tax_report"/>
    </record>
    <record id="tax_report_line_zero_rated" model="account.tax.report.line">
        <field name="name">Zero Rated</field>
        <field name="tag_name">Zero Rated</field>
        <field name="parent_id" ref="tax_report_line_gst_others"/>
        <field name="sequence">7</field>
        <field name="report_id" ref="tax_report"/>
    </record>


    <!-- GST TAXES-->

    <!-- CESS Tax -->

    <record id="cess_sale_5" model="account.tax.template">
        <field name="name">CESS Sale 5%</field>
        <field name="description">CESS 5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'plus_report_line_ids': [ref('tax_report_line_cess')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'minus_report_line_ids': [ref('tax_report_line_cess')],
            }),
        ]"/>
    </record>

    <record id="cess_sale_1591" model="account.tax.template">
        <field name="name">CESS Sale 1591 Per Thousand</field>
        <field name="description">1591 PER THOUSAND</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">fixed</field>
        <field name="amount">1.591</field>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'plus_report_line_ids': [ref('tax_report_line_cess')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'minus_report_line_ids': [ref('tax_report_line_cess')],
            }),
        ]"/>
    </record>

    <record id="cess_5_plus_1591_sale" model="account.tax.template">
        <field name="name">CESS 5% + RS.1591/THOUSAND</field>
        <field name="description">CESS 5% + RS.1591/THOUSAND</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">group</field>
        <field name="amount">0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('cess_sale_5'),ref('cess_sale_1591'),])]"/>
        <field name="tax_group_id" ref="cess_group"/>
    </record>

    <record id="cess_21_4170_higer_sale" model="account.tax.template">
        <field name="name">CESS 21% or RS.4170/THOUSAND</field>
        <field name="description">CESS 21% or RS.4170/THOUSAND</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">code</field>
        <field name="amount">0</field>
        <field name="python_compute">result=base_amount * 0.21
tax=quantity * 4.17
if tax > result:result=tax</field>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'plus_report_line_ids': [ref('tax_report_line_cess')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'minus_report_line_ids': [ref('tax_report_line_cess')],
            }),
        ]"/>
    </record>

    <!-- Exempt -->

    <record id="exempt_sale" model="account.tax.template">
        <field name="name">Exempt Sale</field>
        <field name="description">Exempt</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="exempt_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_exempt')],

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
                'minus_report_line_ids': [ref('tax_report_line_exempt')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',

            }),
        ]"/>
    </record>

    <!-- Nil Rated -->

    <record id="nil_rated_sale" model="account.tax.template">
        <field name="name">Nil Rated</field>
        <field name="description">Nil Rated</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="nil_rated_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_nil_rated')],

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
                'minus_report_line_ids': [ref('tax_report_line_nil_rated')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',

            }),
        ]"/>
    </record>

    <!-- IGST -->

    <record id="igst_sale_0" model="account.tax.template">
        <field name="name">IGST 0%</field>
        <field name="description">IGST 0%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_zero_rated')],
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
                'minus_report_line_ids': [ref('tax_report_line_zero_rated')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="igst_sale_1" model="account.tax.template">
        <field name="name">IGST 1%</field>
        <field name="description">IGST 1%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'plus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'minus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
    </record>

    <record id="igst_sale_2" model="account.tax.template">
        <field name="name">IGST 2%</field>
        <field name="description">IGST 2%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">2</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'plus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'minus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
    </record>

    <record id="igst_sale_28" model="account.tax.template">
        <field name="name">IGST 28%</field>
        <field name="description">IGST 28%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">28</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'plus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'minus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
    </record>

    <record id="igst_sale_18" model="account.tax.template">
        <field name="name">IGST 18%</field>
        <field name="description">IGST 18%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">18</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'plus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'minus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
    </record>

    <record id="igst_sale_12" model="account.tax.template">
        <field name="name">IGST 12%</field>
        <field name="description">IGST 12%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">12</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'plus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'minus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
    </record>

    <record id="igst_sale_5" model="account.tax.template">
        <field name="name">IGST 5%</field>
        <field name="description">IGST 5%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'plus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'minus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
    </record>

    <!-- SGST & CGST Sales Group Tax -->

    <record id="sgst_sale_0_5" model="account.tax.template">
        <field name="name">SGST Sale 0.5%</field>
        <field name="description">SGST 0.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">0.5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'plus_report_line_ids': [ref('tax_report_line_sgst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'minus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
    </record>

    <record id="cgst_sale_0_5" model="account.tax.template">
        <field name="name">CGST Sale 0.5%</field>
        <field name="description">CGST 0.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">0.5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'plus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'minus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
    </record>

    <record id="sgst_sale_1" model="account.tax.template">
        <field name="name">GST 1%</field>
        <field name="description">GST 1%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">group</field>
        <field name="amount">1.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_sale_0_5'), ref('cgst_sale_0_5'),])]"/>
    </record>

    <record id="sgst_sale_1_2" model="account.tax.template">
        <field name="name">SGST Sale 1%</field>
        <field name="description">SGST 1%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'plus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'minus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
    </record>

    <record id="cgst_sale_1_2" model="account.tax.template">
        <field name="name">CGST Sale 1%</field>
        <field name="description">CGST 1%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'plus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'minus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
    </record>

    <record id="sgst_sale_2" model="account.tax.template">
        <field name="name">GST 2%</field>
        <field name="description">GST 2%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">group</field>
        <field name="amount">2.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_sale_1_2'), ref('cgst_sale_1_2'),])]"/>
    </record>

    <record id="sgst_sale_14" model="account.tax.template">
        <field name="name">SGST Sale 14%</field>
        <field name="description">SGST 14%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">14</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'plus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'minus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
    </record>

    <record id="cgst_sale_14" model="account.tax.template">
        <field name="name">CGST Sale 14%</field>
        <field name="description">CGST 14%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">14</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'plus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'minus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
    </record>

    <record id="sgst_sale_28" model="account.tax.template">
        <field name="name">GST 28%</field>
        <field name="description">GST 28%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">group</field>
        <field name="amount">28.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_sale_14'),ref('cgst_sale_14'),])]"/>
    </record>

    <record id="sgst_sale_9" model="account.tax.template">
        <field name="name">SGST Sale 9%</field>
        <field name="description">SGST 9%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">9</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'plus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'minus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
    </record>

    <record id="cgst_sale_9" model="account.tax.template">
        <field name="name">CGST Sale 9%</field>
        <field name="description">CGST 9%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">9</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'plus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'minus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
    </record>

    <record id="sgst_sale_18" model="account.tax.template">
        <field name="name">GST 18%</field>
        <field name="description">GST 18%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">group</field>
        <field name="amount">18.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_sale_9'),ref('cgst_sale_9'),])]"/>
    </record>

     <record id="sgst_sale_6" model="account.tax.template">
        <field name="name">SGST Sale 6%</field>
        <field name="description">SGST 6%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">6</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'plus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'minus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
    </record>

    <record id="cgst_sale_6" model="account.tax.template">
        <field name="name">CGST Sale 6%</field>
        <field name="description">CGST 6%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">6</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'plus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'minus_report_line_ids': [ref('tax_report_line_cgst')],
            }),
        ]"/>
    </record>

    <record id="sgst_sale_12" model="account.tax.template">
        <field name="name">GST 12%</field>
        <field name="description">GST 12%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">group</field>
        <field name="amount">12.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_sale_6'),ref('cgst_sale_6'),])]"/>
    </record>

    <record id="sgst_sale_2_5" model="account.tax.template">
        <field name="name">SGST Sale 2.5%</field>
        <field name="description">SGST 2.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">2.5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'plus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'minus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
    </record>

    <record id="cgst_sale_2_5" model="account.tax.template">
        <field name="name">CGST Sale 2.5%</field>
        <field name="description">CGST 2.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">2.5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'plus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'minus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
    </record>

    <record id="sgst_sale_5" model="account.tax.template">
        <field name="name">GST 5%</field>
        <field name="description">GST 5%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">group</field>
        <field name="amount">5.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="sequence">0</field>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_sale_2_5'), ref('cgst_sale_2_5'),])]"/>
    </record>

    <!-- Purchase Taxes-->

    <!-- CESS Taxes -->

    <record id="cess_purchase_5" model="account.tax.template">
        <field name="name">CESS Purchase 5%</field>
        <field name="description">CESS 5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'minus_report_line_ids': [ref('tax_report_line_cess')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'plus_report_line_ids': [ref('tax_report_line_cess')],
            }),
        ]"/>
    </record>

    <record id="cess_purchase_1591" model="account.tax.template">
        <field name="name">CESS Purchase 1591 Per Thousand</field>
        <field name="description">1591 PER THOUSAND</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">fixed</field>
        <field name="amount">1.591</field>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'minus_report_line_ids': [ref('tax_report_line_cess')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'plus_report_line_ids': [ref('tax_report_line_cess')],
            }),
        ]"/>
    </record>

    <record id="cess_5_plus_1591_purchase" model="account.tax.template">
        <field name="name">CESS 5% + RS.1591/THOUSAND</field>
        <field name="description">CESS 5% + RS.1591/THOUSAND</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('cess_purchase_5'),ref('cess_purchase_1591'),])]"/>
        <field name="tax_group_id" ref="cess_group"/>
    </record>

    <record id="cess_21_4170_higer_purchase" model="account.tax.template">
        <field name="name">CESS 21% or RS.4170/THOUSAND</field>
        <field name="description">CESS 21% or RS.4170/THOUSAND</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">code</field>
        <field name="amount">0</field>
        <field name="python_compute">result=base_amount * 0.21
tax=quantity * 4.17
if tax > result:result=tax</field>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'minus_report_line_ids': [ref('tax_report_line_cess')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'plus_report_line_ids': [ref('tax_report_line_cess')],
            }),
        ]"/>
    </record>

    <!-- Exempt -->

    <record id="exempt_purchase" model="account.tax.template">
        <field name="name">Exempt purchase</field>
        <field name="description">Exempt</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="exempt_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_exempt')],

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
                'plus_report_line_ids': [ref('tax_report_line_exempt')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <!-- Nil Rated -->

    <record id="nil_rated_purchase" model="account.tax.template">
        <field name="name">Nil Rated</field>
        <field name="description">Nil Rat</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="nil_rated_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_nil_rated')],

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
                'plus_report_line_ids': [ref('tax_report_line_nil_rated')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',

            }),
        ]"/>
    </record>

    <!-- IGST -->

    <record id="igst_purchase_0" model="account.tax.template">
        <field name="name">IGST 0%</field>
        <field name="description">IGST 0%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_line_zero_rated')],
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
                'plus_report_line_ids': [ref('tax_report_line_zero_rated')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="igst_purchase_1" model="account.tax.template">
        <field name="name">IGST 1%</field>
        <field name="description">IGST 1%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'minus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'plus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
    </record>

    <record id="igst_purchase_2" model="account.tax.template">
        <field name="name">IGST 2%</field>
        <field name="description">IGST 2%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">2</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'minus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'plus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
    </record>

    <record id="igst_purchase_28" model="account.tax.template">
        <field name="name">IGST 28%</field>
        <field name="description">IGST 28%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">28</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'minus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'plus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
    </record>

    <record id="igst_purchase_18" model="account.tax.template">
        <field name="name">IGST 18%</field>
        <field name="description">IGST 18%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">18</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'minus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'plus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
    </record>

    <record id="igst_purchase_12" model="account.tax.template">
        <field name="name">IGST 12%</field>
        <field name="description">IGST 12%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">12</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'minus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'plus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
    </record>

    <record id="igst_purchase_5" model="account.tax.template">
        <field name="name">IGST 5%</field>
        <field name="description">IGST 5%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'minus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'plus_report_line_ids': [ref('tax_report_line_igst')],
            }),
        ]"/>
    </record>

    <!-- SGST & CGST -->

    <record id="sgst_purchase_0_5" model="account.tax.template">
        <field name="name">SGST Purchase 0.5%</field>
        <field name="description">SGST 0.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">0.5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'minus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'plus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
    </record>

    <record id="cgst_purchase_0_5" model="account.tax.template">
        <field name="name">CGST Purchase 0.5%</field>
        <field name="description">CGST 0.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">0.5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'minus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'plus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
    </record>

    <record id="sgst_purchase_1" model="account.tax.template">
        <field name="name">GST 1%</field>
        <field name="description">GST 1%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">1.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_purchase_0_5'),ref('cgst_purchase_0_5'),])]"/>
    </record>

    <record id="sgst_purchase_1_2" model="account.tax.template">
        <field name="name">SGST Purchase 1%</field>
        <field name="description">SGST 1%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'minus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'plus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
    </record>

    <record id="cgst_purchase_1_2" model="account.tax.template">
        <field name="name">CGST Purchase 1%</field>
        <field name="description">CGST 1%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'minus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'plus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>

    </record>

    <record id="sgst_purchase_2" model="account.tax.template">
        <field name="name">GST 2%</field>
        <field name="description">GST 2%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">2.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_purchase_1_2'),ref('cgst_purchase_1_2'),])]"/>
    </record>

    <record id="sgst_purchase_14" model="account.tax.template">
        <field name="name">SGST Purchase 14%</field>
        <field name="description">SGST 14%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">14</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'minus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'plus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
    </record>

    <record id="cgst_purchase_14" model="account.tax.template">
        <field name="name">CGST Purchase 14%</field>
        <field name="description">CGST 14%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">14</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'minus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'plus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
    </record>

    <record id="sgst_purchase_28" model="account.tax.template">
        <field name="name">GST 28%</field>
        <field name="description">GST 28%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">28.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_purchase_14'),ref('cgst_purchase_14'),])]"/>
    </record>

    <record id="sgst_purchase_9" model="account.tax.template">
        <field name="name">SGST Purchase 9%</field>
        <field name="description">SGST 9%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">9</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'minus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'plus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
    </record>

    <record id="cgst_purchase_9" model="account.tax.template">
        <field name="name">CGST Purchase 9%</field>
        <field name="description">CGST 9%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">9</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'minus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'plus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
    </record>

    <record id="sgst_purchase_18" model="account.tax.template">
        <field name="name">GST 18%</field>
        <field name="description">GST 18%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">18.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_purchase_9'),ref('cgst_purchase_9'),])]"/>
    </record>

    <record id="sgst_purchase_6" model="account.tax.template">
        <field name="name">SGST Purchase 6%</field>
        <field name="description">SGST 6%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">6</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'minus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'plus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
    </record>

    <record id="cgst_purchase_6" model="account.tax.template">
        <field name="name">CGST Purchase 6%</field>
        <field name="description">CGST 6%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">6</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'minus_report_line_ids': [ref('tax_report_line_cgst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'plus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
    </record>

    <record id="sgst_purchase_12" model="account.tax.template">
        <field name="name">GST 12%</field>
        <field name="description">GST 12%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">12.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_purchase_6'),ref('cgst_purchase_6'),])]"/>
    </record>

    <record id="sgst_purchase_2_5" model="account.tax.template">
        <field name="name">SGST Purchase 2.5%</field>
        <field name="description">SGST 2.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">2.5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'minus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'plus_report_line_ids': [ref('tax_report_line_sgst')],
            })
        ]"/>
    </record>

    <record id="cgst_purchase_2_5" model="account.tax.template">
        <field name="name">CGST Purchase 2.5%</field>
        <field name="description">CGST 2.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">2.5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'minus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'plus_report_line_ids': [ref('tax_report_line_cgst')],
            })
        ]"/>
    </record>

    <record id="sgst_purchase_5" model="account.tax.template">
        <field name="name">GST 5%</field>
        <field name="description">GST 5%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">5.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="sequence">0</field>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_purchase_2_5'),ref('cgst_purchase_2_5'),])]"/>
    </record>

    <!-- Purchase Reverse Charge Taxes-->

    <!-- CESS Taxes -->

    <record id="cess_purchase_5_rc" model="account.tax.template">
        <field name="name">CESS Purchase 5% (RC)</field>
        <field name="description">CESS 5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'minus_report_line_ids': [ref('tax_report_line_cess_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'plus_report_line_ids': [ref('tax_report_line_cess_rc')],
            }),
        ]"/>
    </record>

    <record id="cess_purchase_1591_rc" model="account.tax.template">
        <field name="name">CESS Purchase 1591 Per Thousand (RC)</field>
        <field name="description">1591 PER THOUSAND</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">fixed</field>
        <field name="amount">1.591</field>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'minus_report_line_ids': [ref('tax_report_line_cess_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'plus_report_line_ids': [ref('tax_report_line_cess_rc')],
            }),
        ]"/>
    </record>

    <record id="cess_5_plus_1591_purchase_rc" model="account.tax.template">
        <field name="name">CESS 5% + RS.1591/THOUSAND (RC)</field>
        <field name="description">CESS 5% + RS.1591/THOUSAND</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('cess_purchase_5_rc'),ref('cess_purchase_1591_rc'),])]"/>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
    </record>

    <record id="cess_21_4170_higer_purchase_rc" model="account.tax.template">
        <field name="name">CESS 21% or RS.4170/THOUSAND (RC)</field>
        <field name="description">CESS 21% or RS.4170/THOUSAND</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">code</field>
        <field name="amount">0</field>
        <field name="python_compute">result=base_amount * 0.21
tax=quantity * 4.17
if tax > result:result=tax</field>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'minus_report_line_ids': [ref('tax_report_line_cess_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'plus_report_line_ids': [ref('tax_report_line_cess_rc')],
            }),
        ]"/>
    </record>


    <!-- IGST -->

    <record id="igst_purchase_1_rc" model="account.tax.template">
        <field name="name">IGST 1% (RC)</field>
        <field name="description">IGST 1%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'minus_report_line_ids': [ref('tax_report_line_igst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'plus_report_line_ids': [ref('tax_report_line_igst'), ref('tax_report_line_igst_rc')],
            }),
        ]"/>
    </record>

    <record id="igst_purchase_2_rc" model="account.tax.template">
        <field name="name">IGST 2% (RC)</field>
        <field name="description">IGST 2%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">2</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'minus_report_line_ids': [ref('tax_report_line_igst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'plus_report_line_ids': [ref('tax_report_line_igst_rc')],
            }),
        ]"/>
    </record>

    <record id="igst_purchase_28_rc" model="account.tax.template">
        <field name="name">IGST 28% (RC)</field>
        <field name="description">IGST 28%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">28</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'minus_report_line_ids': [ref('tax_report_line_igst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'plus_report_line_ids': [ref('tax_report_line_igst'), ref('tax_report_line_igst_rc')],
            }),
        ]"/>
    </record>

    <record id="igst_purchase_18_rc" model="account.tax.template">
        <field name="name">IGST 18% (RC)</field>
        <field name="description">IGST 18%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">18</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'minus_report_line_ids': [ref('tax_report_line_igst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'plus_report_line_ids': [ref('tax_report_line_igst_rc')],
            }),
        ]"/>
    </record>

    <record id="igst_purchase_12_rc" model="account.tax.template">
        <field name="name">IGST 12% (RC)</field>
        <field name="description">IGST 12%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">12</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'minus_report_line_ids': [ref('tax_report_line_igst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'plus_report_line_ids': [ref('tax_report_line_igst_rc')],
            }),
        ]"/>
    </record>

    <record id="igst_purchase_5_rc" model="account.tax.template">
        <field name="name">IGST 5% (RC)</field>
        <field name="description">IGST 5%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'minus_report_line_ids': [ref('tax_report_line_igst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'plus_report_line_ids': [ref('tax_report_line_igst_rc')],
            }),
        ]"/>
    </record>

    <!-- SGST & CGST -->

    <record id="sgst_purchase_0_5_rc" model="account.tax.template">
        <field name="name">SGST Purchase 0.5% (RC)</field>
        <field name="description">SGST 0.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">0.5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'minus_report_line_ids': [ref('tax_report_line_sgst_rc')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'plus_report_line_ids': [ref('tax_report_line_sgst_rc')],
            })
        ]"/>
    </record>

    <record id="cgst_purchase_0_5_rc" model="account.tax.template">
        <field name="name">CGST Purchase 0.5% (RC)</field>
        <field name="description">CGST 0.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">0.5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'minus_report_line_ids': [ref('tax_report_line_cgst_rc')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'plus_report_line_ids': [ref('tax_report_line_cgst_rc')],
            })
        ]"/>
    </record>

    <record id="sgst_purchase_1_rc" model="account.tax.template">
        <field name="name">GST 1% (RC)</field>
        <field name="description">GST 1%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">1.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_purchase_0_5_rc'),ref('cgst_purchase_0_5_rc'),])]"/>
    </record>

    <record id="sgst_purchase_1_2_rc" model="account.tax.template">
        <field name="name">SGST Purchase 1% (RC)</field>
        <field name="description">SGST 1%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'minus_report_line_ids': [ref('tax_report_line_sgst_rc')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'plus_report_line_ids': [ref('tax_report_line_sgst_rc')],
            })
        ]"/>
    </record>

    <record id="cgst_purchase_1_2_rc" model="account.tax.template">
        <field name="name">CGST Purchase 1% (RC)</field>
        <field name="description">CGST 1%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'minus_report_line_ids': [ref('tax_report_line_cgst_rc')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'plus_report_line_ids': [ref('tax_report_line_cgst_rc')],
            })
        ]"/>

    </record>

    <record id="sgst_purchase_2_rc" model="account.tax.template">
        <field name="name">GST 2% (RC)</field>
        <field name="description">GST 2%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">2.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_purchase_1_2_rc'),ref('cgst_purchase_1_2_rc'),])]"/>
    </record>

    <record id="sgst_purchase_14_rc" model="account.tax.template">
        <field name="name">SGST Purchase 14% (RC)</field>
        <field name="description">SGST 14%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">14</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'minus_report_line_ids': [ref('tax_report_line_sgst_rc')],
            }),

        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'plus_report_line_ids': [ref('tax_report_line_sgst_rc')],
            }),
        ]"/>
    </record>

    <record id="cgst_purchase_14_rc" model="account.tax.template">
        <field name="name">CGST Purchase 14% (RC)</field>
        <field name="description">CGST 14%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">14</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'minus_report_line_ids': [ref('tax_report_line_cgst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'plus_report_line_ids': [ref('tax_report_line_cgst_rc')],
            }),
        ]"/>
    </record>

    <record id="sgst_purchase_28_rc" model="account.tax.template">
        <field name="name">GST 28% (RC)</field>
        <field name="description">GST 28%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">28.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_purchase_14_rc'),ref('cgst_purchase_14_rc'),])]"/>
    </record>

    <record id="sgst_purchase_9_rc" model="account.tax.template">
        <field name="name">SGST Purchase 9% (RC)</field>
        <field name="description">SGST 9%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">9</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'minus_report_line_ids': [ref('tax_report_line_sgst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'plus_report_line_ids': [ref('tax_report_line_sgst_rc')],
            }),
        ]"/>
    </record>

    <record id="cgst_purchase_9_rc" model="account.tax.template">
        <field name="name">CGST Purchase 9% (RC)</field>
        <field name="description">CGST 9%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">9</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'minus_report_line_ids': [ref('tax_report_line_cgst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'plus_report_line_ids': [ref('tax_report_line_cgst_rc')],
            }),
        ]"/>
    </record>

    <record id="sgst_purchase_18_rc" model="account.tax.template">
        <field name="name">GST 18% (RC)</field>
        <field name="description">GST 18%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">18.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_purchase_9_rc'),ref('cgst_purchase_9_rc'),])]"/>
    </record>

    <record id="sgst_purchase_6_rc" model="account.tax.template">
        <field name="name">SGST Purchase 6% (RC)</field>
        <field name="description">SGST 6%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">6</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'minus_report_line_ids': [ref('tax_report_line_sgst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'plus_report_line_ids': [ref('tax_report_line_sgst_rc')],
            }),
        ]"/>
    </record>

    <record id="cgst_purchase_6_rc" model="account.tax.template">
        <field name="name">CGST Purchase 6% (RC)</field>
        <field name="description">CGST 6%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">6</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'minus_report_line_ids': [ref('tax_report_line_cgst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'plus_report_line_ids': [ref('tax_report_line_cgst_rc')],
            }),
        ]"/>
    </record>

    <record id="sgst_purchase_12_rc" model="account.tax.template">
        <field name="name">GST 12% (RC)</field>
        <field name="description">GST 12%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">12.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_purchase_6_rc'),ref('cgst_purchase_6_rc'),])]"/>
    </record>

    <record id="sgst_purchase_2_5_rc" model="account.tax.template">
        <field name="name">SGST Purchase 2.5% (RC)</field>
        <field name="description">SGST 2.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">2.5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'minus_report_line_ids': [ref('tax_report_line_sgst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'plus_report_line_ids': [ref('tax_report_line_sgst_rc')],
            })
        ]"/>
    </record>

    <record id="cgst_purchase_2_5_rc" model="account.tax.template">
        <field name="name">CGST Purchase 2.5% (RC)</field>
        <field name="description">CGST 2.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">2.5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'minus_report_line_ids': [ref('tax_report_line_cgst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'plus_report_line_ids': [ref('tax_report_line_cgst_rc')],
            }),
        ]"/>
    </record>

    <record id="sgst_purchase_5_rc" model="account.tax.template">
        <field name="name">GST 5% (RC)</field>
        <field name="description">GST 5%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">5.0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="gst_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="sequence">0</field>
        <field name="children_tax_ids" eval="[(6, 0, [ref('sgst_purchase_2_5_rc'),ref('cgst_purchase_2_5_rc'),])]"/>
    </record>

</odoo>

```

## File: data\l10n_in.port.code.csv

```csv
id,state_id:id,code,name
port_code_incnb1,base.state_in_an,INCNB1,Car-Nicobar
port_code_incrn1,base.state_in_an,INCRN1,Cornwallis
port_code_inmyb1,base.state_in_an,INMYB1,Mayabandar
port_code_inesh1,base.state_in_an,INESH1,Elphinstone Harbour
port_code_inrgt1,base.state_in_an,INRGT1,Ranghat Bay
port_code_inmdw1,base.state_in_an,INMDW1,Meadows
port_code_innan1,base.state_in_an,INNAN1,Nancowrie
port_code_inixz1,base.state_in_an,INIXZ1,Port Blair
port_code_inixz4,base.state_in_an,INIXZ4,Port Blair
port_code_inhyd4,base.state_in_ap,INHYD4,Hyderabad Air Cargo
port_code_inbnp1,base.state_in_ap,INBNP1,Bheemunipatnam
port_code_innvp1,base.state_in_ap,INNVP1,Navaspur
port_code_invtz1,base.state_in_ap,INVTZ1,Vizac Sea
port_code_insrv1,base.state_in_ap,INSRV1,Surasani – Yanam
port_code_invtz6,base.state_in_ap,INVTZ6,Visakhapatnam (EPZ/SEZ)
port_code_invru1,base.state_in_ap,INVRU1,Vadarevu
port_code_inkak1,base.state_in_ap,INKAK1,Kakinada
port_code_inrpl6,base.state_in_ap,INRPL6,Raddipalem
port_code_inkdd6,base.state_in_ap,INKDD6,Karedu
port_code_invtz4,base.state_in_ap,INVTZ4,Vishakapatnam
port_code_inapt6,base.state_in_ap,INAPT6,Anaparthi
port_code_inclx6,base.state_in_ap,INCLX6,Chirala
port_code_insnf6,base.state_in_ap,INSNF6,Hyderabad
port_code_incoi6,base.state_in_ap,INCOI6,Kakinada
port_code_inmap1,base.state_in_ap,INMAP1,Masulipatnam
port_code_inkri6,base.state_in_ap,INKRI1,Krishnapatnam
port_code_inakp6,base.state_in_ap,INAKP6,APIICL SEZ/Visakhapatnam
port_code_intni6,base.state_in_ap,INTNI6,HIPL SEZ/Visakhapatnam
port_code_inakr6,base.state_in_ap,INAKR6,RPCIPL SEZ/Visakhapatnam
port_code_invzm6,base.state_in_ap,INVZM6,DLL SEZ/Visakhapatnam
port_code_inakb6,base.state_in_ap,INAKB6,BIACPL SEZ/Visakhapatnam
port_code_innrp6,base.state_in_ap,INNRP6,AAL-SEZ/Visakhapatnam
port_code_inmov6,base.state_in_ap,INMOV6,VBTL-SEZ/Medak
port_code_inkvr6,base.state_in_ap,INKVR6,WFPML-SEZ/KOVVUR
port_code_infma6,base.state_in_ap,INFMA6,APIICL/Medak District
port_code_infmh6,base.state_in_ap,INFMH6,hgsezl/Ranga Reddy
port_code_incoa6,base.state_in_ap,INCOA6,KSPL-SEZ/KAKINADA
port_code_ingly6,base.state_in_ap,INGLY6,APIICL-SEZ/MAHABOOBNAGAR
port_code_inspe6,base.state_in_ap,INSPE6,ASDIPL-SEZ/NELLORE
port_code_incop6,base.state_in_ap,INCOP6,PICPL-SEZ/KAKINADA
port_code_inmde6,base.state_in_ap,INMDE6,APIICL SEZ/MEDAK
port_code_inkoh6,base.state_in_ap,INKOH6,RLL-SEZ/Medak
port_code_inurf6,base.state_in_ap,INURF6,FAB CITY SPV Distt. Ranga Reddy
port_code_inmea6,base.state_in_ap,INMEA6,APIIC-SEZ/Vill-Lalgadi Distt.-Ranga
port_code_indbs6,base.state_in_ap,INDBS6,SANTA-SEZ/Vill-Muppireddipally
port_code_inurg6,base.state_in_ap,INURG6,GMR Hyderabad
port_code_intas6,base.state_in_ap,INTAS6,Sri City Private Limited
port_code_ingnr6,base.state_in_ap,INGNR6,LIPL-ICD/Marripalem, Guntur
port_code_infms6,base.state_in_ap,INFMS6,Stargaze/Rangareddy/SEZ
port_code_infmj6,base.state_in_ap,INFMJ6,J T/SEZ/Rangareddy
port_code_inlpi6,base.state_in_ap,INLPI6,Sundew/SEZ/Rangareddy
port_code_inlpd6,base.state_in_ap,INLPD6,DLF/SEZ/Rangareddy
port_code_intmi6,base.state_in_ap,INTMI6,IIFFCO/SEZ/Nellore
port_code_inhur6,base.state_in_ap,INHUR6,Rassi/SEZ/Anantpur
port_code_invzr6,base.state_in_ap,INVZR6,Reddy’r/SEZ/Srikakulam
port_code_incdp6,base.state_in_ap,INCDP6,APIIC/SEZ/Cuddapah
port_code_intmx6,base.state_in_ap,INTMX6,M/s CONTINENTAL MULTIMODAL
port_code_innpgb,base.state_in_ar,INNPGB,TNEaRmMpIoNnAgLS LTD.
port_code_indrgb,base.state_in_as,INDRGB,Darranga
port_code_ingau4,base.state_in_as,INGAU4,Gauhati
port_code_inghwb,base.state_in_as,INGHWB,Gauhati Steamerghat
port_code_indhbb,base.state_in_as,INDHBB,Dhubri Steamerghat
port_code_ingtgb,base.state_in_as,INGTGB,Gitaldah road
port_code_ingkj2,base.state_in_as,INGKJ2,Golakganj Raiiway Station
port_code_inhtsb,base.state_in_as,INHTSB,Hatisar
port_code_inkgjb,base.state_in_as,INKGJB,Karimganj Steamerghat and Ferry
port_code_instrb,base.state_in_as,INSTRB,Sutarkandi
port_code_inslrb,base.state_in_as,INSLRB,Silcher Steamerghat
port_code_inslr2,base.state_in_as,INSLR2,Silcher R.M.S. Office
port_code_inphbb,base.state_in_as,INPHBB,Phulbari
port_code_innmtb,base.state_in_as,INNMTB,Neamati steamer Ghat
port_code_inmnub,base.state_in_as,INMNUB,Manu
port_code_inmkcb,base.state_in_as,INMKCB,Manikarchar
port_code_inmhn2,base.state_in_as,INMHN2,Mahisashan Railway Station
port_code_inltbb,base.state_in_as,INLTBB,Latu Bazar
port_code_inulpb,base.state_in_as,INULPB,Ultapani
port_code_intjpb,base.state_in_as,INTJPB,Tezpur Steamerghat
port_code_inkxj2,base.state_in_as,INKXJ2,Karimganj Railway Station
port_code_inhld2,base.state_in_as,INHLD2,Haldibari Railway Station
port_code_ingkjb,base.state_in_as,INGKJB,Golakganj (LCS)
port_code_inamg6,base.state_in_as,INAMG6,Amingaon(Gauhati)
port_code_inrxl8,base.state_in_br,INRXL8,Rexaul
port_code_ingalb,base.state_in_br,INGALB,Galgalia
port_code_injbnb,base.state_in_br,INJBNB,Jogbani
port_code_insnbb,base.state_in_br,INSNBB,Sonabarsa
port_code_inrxlb,base.state_in_br,INRXLB,Raxaul
port_code_inknlb,base.state_in_br,INKNLB ,Kunauli
port_code_inktrb,base.state_in_br,INKTRB,Kathihar
port_code_injayb,base.state_in_br,INJAYB,Jayanagar
port_code_inbtmb,base.state_in_br,INBTMB,Bhithamore(Sursnad)
port_code_inbgub,base.state_in_br,INBGUB,Bairgania
port_code_inbnrb,base.state_in_br,INBNRB,Bhimnagar
port_code_ingay4,base.state_in_br,INGAY4,Gaya
port_code_inpat4,base.state_in_br,INPAT4,Patna
port_code_inrai6,base.state_in_cg,INRAI6,Raipur
port_code_inrjn6,base.state_in_cg,INRJN6,M/s LANCO SOLAR PRIVATE LTD.
port_code_indam1,base.state_in_dd,INDAM1,Daman & Diu
port_code_inshp1,base.state_in_dd,INSHP1,Sinbhour Port
port_code_indel4,base.state_in_dl,INDEL4,Delhi Air Cargo
port_code_indli2,base.state_in_dl,INDLI2,Delhi Railway Station
port_code_intkd6,base.state_in_dl,INTKD6,Tughlakabad
port_code_inppg6,base.state_in_dl,INPPG6,Patparganj
port_code_inmrm1,base.state_in_ga,INMRM1,Goa Sea
port_code_intpn1,base.state_in_ga,INTPN1,Talpona
port_code_inppj1,base.state_in_ga,INPPJ1,Pellet Plant Jetty at Shiroda
port_code_inpnj1,base.state_in_ga,INPNJ1,Panjim
port_code_inchr1,base.state_in_ga,INCHR1,Chapora
port_code_inbet1,base.state_in_ga,INBET1,Betul
port_code_inmdg6,base.state_in_ga,INMDG6,Margao
port_code_ingoi4,base.state_in_ga,INGOI4,Dabolim
port_code_inpan1,base.state_in_ga,INPAN1,Panaji Port
port_code_insbi6,base.state_in_gj,INSBI6,Sabarmati ICD
port_code_inpav1,base.state_in_gj,INPAV1,Pipavav(Victor) Port
port_code_inbed1,base.state_in_gj,INBED1,Bedi(Including Rozi-Jamnagar)
port_code_inbgw1,base.state_in_gj,INBGW1,Bhagwa
port_code_inkdn1,base.state_in_gj,INKDN1,Kodinar(Muldwarka)
port_code_inakvi1,base.state_in_gj,INKVI1,Kavi
port_code_ingha1,base.state_in_gj,INGHA1,Ghogha
port_code_ingin6,base.state_in_gj,INGIN6,Gandhidham
port_code_indrk1,base.state_in_gj,INDRK1,Dwarka (Rupen)
port_code_indhr1,base.state_in_gj,INDHR1,Dholera
port_code_incmb1,base.state_in_gj,INCMB1,Cambay
port_code_inbsr1,base.state_in_gj,INBSR1,Bulsar
port_code_inblm1,base.state_in_gj,INBLM1,Bilimora
port_code_inhza6,base.state_in_gj,INHZA6,Hazira SEZ / Surat
port_code_inbhd6,base.state_in_gj,INBHD6,Dahez SEZ
port_code_inlpj6,base.state_in_gj,INLPJ6,Reliance SEZ/ Jamnagar
port_code_inbco6,base.state_in_gj,INBCO6,Euro Multivision Bhachau SEZ / Kutch
port_code_inaje6,base.state_in_gj,INAJE6,Welspun Anjar SEZ / Anjar
port_code_injbl6,base.state_in_gj,INJBL6,E-Complex SEZ / Amreli
port_code_inkdl6,base.state_in_gj,INKDL6,Kandla SEZ / Gandhidham
port_code_inval6,base.state_in_gj,INVAL6,Valvada ICD
port_code_insbh1,base.state_in_gj,INSBH1,Sinbhour
port_code_insmr1,base.state_in_gj,INSMR1,Simor
port_code_inrjp1,base.state_in_gj,INRJP1,Rajpara
port_code_inonj1,base.state_in_gj,INONJ1,Onjal
port_code_inomu1,base.state_in_gj,INOMU1,Old Mundra
port_code_innvb1,base.state_in_gj,INNVB1,Navabunder
port_code_inmtw1,base.state_in_gj,INMTW1,Metwad
port_code_inktw1,base.state_in_gj,INKTW1,Koteshwar
port_code_inktd1,base.state_in_gj,INKTD1,Kotda
port_code_inumr1,base.state_in_gj,INUMR1,Umarsadi
port_code_intnk1,base.state_in_gj,INTNK1,T ankari
port_code_inbyt1,base.state_in_gj,INBYT1,Beyt
port_code_inada6,base.state_in_gj,INADA6,Adalaj
port_code_inbhu1,base.state_in_gj,INBHU1,Bhavanagar
port_code_indiv1,base.state_in_gj,INDIV1,Div
port_code_inixy1,base.state_in_gj,INIXY1,Kandla
port_code_injbd1,base.state_in_gj,INJBD1,Jafrabad
port_code_inumb1,base.state_in_gj,INUMB1,Umbergoan
port_code_intun1,base.state_in_gj,INTUN1,Tuna
port_code_intja1,base.state_in_gj,INTJA1,Talaja
port_code_insik1,base.state_in_gj,INSIK1,Sikka
port_code_insal1,base.state_in_gj,INSAL1,Salaya
port_code_inpin1,base.state_in_gj,INPIN1,Pindhara
port_code_inpbd1,base.state_in_gj,INPBD1,Porbandar
port_code_inokh1,base.state_in_gj,INOKH1,Okha
port_code_innav1,base.state_in_gj,INNAV1,Navlakhi
port_code_inraj6,base.state_in_gj,INRAJ6,Rajkot
port_code_invpi6,base.state_in_gj,INVPI6,Vapi
port_code_instv6,base.state_in_gj,INSTV6,Surat (EPZ/SEZ)
port_code_inixy6,base.state_in_gj,INIXY6,Kandla (EPZ/SEZ)
port_code_insac6,base.state_in_gj,INSAC6,Sachin(Surat)
port_code_inkap6,base.state_in_gj,INKAP6,Kapadra(Surat)
port_code_inbrc6,base.state_in_gj,INBRC6,Baroda
port_code_inamd5,base.state_in_gj,INAMD5,Amhedabad
port_code_inamd4,base.state_in_gj,INAMD4,Ahmedabad
port_code_inala1,base.state_in_gj,INALA1,ALANG SBY
port_code_inmdv1,base.state_in_gj,INMDV1,Mandvi
port_code_invva1,base.state_in_gj,INVVA1,Veraval
port_code_invsi1,base.state_in_gj,INVSI1,Vansi-Borsi
port_code_invad1,base.state_in_gj,INVAD1,Vadinar
port_code_inmun1,base.state_in_gj,INMUN1,Mundra
port_code_inmli1,base.state_in_gj,INMLI1,Maroli
port_code_inmha1,base.state_in_gj,INMHA1,Mahuva
port_code_inmgr1,base.state_in_gj,INMGR1,Mangrol
port_code_inmdk1,base.state_in_gj,INMDK1,Muldwarka
port_code_inmda1,base.state_in_gj,INMDA1,Magdalla
port_code_inkok1,base.state_in_gj,INKOK1,Koka
port_code_injda1,base.state_in_gj,INJDA1,Jodia
port_code_injak1,base.state_in_gj,INJAK1,Jakhau
port_code_ingga1,base.state_in_gj,INGGA1,Gogha
port_code_indah1,base.state_in_gj,INDAH1,Dahej
port_code_inbrh1,base.state_in_gj,INBRH1,Broach
port_code_inakv6,base.state_in_gj,INAKV6,Ankleshwar
port_code_inchn6,base.state_in_gj,INCHN6,Vadodara – Chhani
port_code_insau6,base.state_in_gj,INSAU6,Thar Dry Port – Ahemdabad ICD
port_code_inudn6,base.state_in_gj,INUDN6,GHB-SEZ/SURAT
port_code_inajm6,base.state_in_gj,INAJM6,Mundra Port SEZ
port_code_insch6,base.state_in_gj,INSCH6,SAP-SEZ/SURAT
port_code_inapi6,base.state_in_gj,INAPI6 ,AAP-SEZ/AHMEDABAD
port_code_ingng6,base.state_in_gj,INGNG6,GIDC-SEZ/GANDHINAGAR
port_code_inadg6,base.state_in_gj,INADG6,GIPL-SEZ/AHMEDABAD
port_code_inadc6,base.state_in_gj,INADC6,CCIPL-SEZ/AHMEDABAD
port_code_inzip6,base.state_in_gj,INZIP6,ZIPL-SEZ/AHMEDABAD
port_code_ingns6,base.state_in_gj,INGNS6,SREHPL-SEZ/GANDHINAGAR
port_code_inadm6,base.state_in_gj,INADM6,MRPL-SEZ/AHMEDABAD
port_code_inadr6,base.state_in_gj,INADR6,CGRPL-SEZ/AHMEDABAD
port_code_ingnt6,base.state_in_gj,INGNT6,TCS-SEZ/GANDHINAGAR
port_code_inbhs6,base.state_in_gj,INBHS6,Sterling – SEZ / Bharuch
port_code_invln6,base.state_in_gj,INVLN6,NG REALTY- SEZ/Taluka Bavla Distt.
port_code_inbrs6,base.state_in_gj,INBRS6,SAhE m&e Cd aLbTaD.d- SEZ /WAGHODIA
port_code_inbhc6,base.state_in_gj,INBHC6,Jubilant-Chemical-SEZ/Vilayat
port_code_invld6,base.state_in_gj,INVLD6,Dishman-Pharmaceutical
port_code_inhzr6,base.state_in_gj,INHZR6,Surat ICD
port_code_inbrl6,base.state_in_gj,INBRL6,L and T ltd./SEZ/Vadodara
port_code_inhza1,base.state_in_gj,INHZA1,HAZIRA PORT/SURAT
port_code_ingna6,base.state_in_gj,INGNA6,APPL SEZ/Gandhinagar
port_code_ingnc6,base.state_in_gj,INGNC6,GIFT SEZ/Gandhinagar
port_code_inkbc6,base.state_in_gj,INKBC6,KRIL ICD / HAZIRA
port_code_ingao6,base.state_in_gj,INGAO6,OPGS SEZ / Gandhidham
port_code_inbdm6,base.state_in_hr,INBDM6,PANCHI GUJARAN, Sonepat ICD
port_code_inghr6,base.state_in_hr,INGHR6,Garhi Harsaru – Gurgaon ICD
port_code_inrea6,base.state_in_hr,INREA6,Rewari
port_code_inptl6,base.state_in_hr,INPTL6,Patli ICD
port_code_innur6,base.state_in_hr,INNUR6,Kundli
port_code_inpnp6,base.state_in_hr,INPNP6,Babarpur
port_code_infbd6,base.state_in_hr,INFBD6,Faridabad
port_code_inbvc6,base.state_in_hr,INBVC6,CONCOR-ICD/BALLABHGARH
port_code_inbfr6,base.state_in_hr,INBFR6,Piyala/Ballabhgarh ICD
port_code_incdd6,base.state_in_hr,INCDD6,DCA-I SEZ/CHANDIGARH
port_code_incdc6,base.state_in_hr,INCDC6,DCA-II SEZ/CHANDIGARH
port_code_inpkr6,base.state_in_hr,INPKR6,KRIL ICD/PALI
port_code_inngsb,base.state_in_hp,INNGSB,Village Namgaya Shipkila
port_code_insxr4,base.state_in_jk,INSXR4,Srinagar
port_code_inbbm6,base.state_in_jk,INBBM6,Bari Brahma
port_code_inixw6,base.state_in_jh,INIXW6,Jamshedpur (ICD)
port_code_intat6,base.state_in_jh,INTAT6,Jamshedpur (ICD)
port_code_inblr4,base.state_in_ka,INBLR4,Banglore Air Cargo
port_code_inblr5,base.state_in_ka,INBLR5,Bangalore
port_code_indru6,base.state_in_ka,INDRU6,Belgaum – Desur
port_code_incdp1,base.state_in_ka,INCDP1,Coondapur (Ganguly)
port_code_inpdd1,base.state_in_ka,INPDD1,Padubidri Minor Port
port_code_inkrw1,base.state_in_ka,INKRW1,Karwar(including Sardeshivagad)
port_code_inhwr1,base.state_in_ka,INHWR1,Honawar
port_code_inhgt1,base.state_in_ka,INHGT1,Hangarkatta
port_code_inbtk1,base.state_in_ka,INBTK1,Bhatkal
port_code_inbkr1,base.state_in_ka,INBKR1,Belekeri
port_code_inbdr1,base.state_in_ka,INBDR1,Baindur
port_code_innml1,base.state_in_ka,INNML1,Mangalore Sea
port_code_inbgq6,base.state_in_ka,INBGQ6,Quest SEZ Belgaum
port_code_inmaq6,base.state_in_ka,INMAQ6,Mangalore SEZ
port_code_inudi6,base.state_in_ka,INUDI6,Synefra SEZ / Udipi
port_code_inbnc6,base.state_in_ka,INBNC6,KBITS SEZ / Bangalore
port_code_inhsp6,base.state_in_ka,INHSP6,KIADBP SEZ / Hassan
port_code_inhsf6,base.state_in_ka,INHSF6,KIADBFP SEZ / Hassan
port_code_inhst6,base.state_in_ka,INHST6,KIADBT SEZ / Hassan
port_code_insbc6,base.state_in_ka,INSBC6,Biocon SEZ / Bangalore
port_code_inblk1,base.state_in_ka,INBLK1,Belekeri
port_code_incoo1,base.state_in_ka,INCOO1,Coondapur (Ganguly)
port_code_inmal1,base.state_in_ka,INMAL1,Malpe
port_code_inhas6,base.state_in_ka,INHAS6,Hassan (EPZ/SEZ)
port_code_inwfd6,base.state_in_ka,INWFD6,Bangalore
port_code_intri1,base.state_in_ka,INTRI1,Tadri
port_code_inixe1,base.state_in_ka,INIXE1,Mangalore
port_code_intrv4,base.state_in_kl,INTRV4,Trivendrun Air Cargo
port_code_inang1,base.state_in_kl,INANG1,Anijengo
port_code_inbdg1,base.state_in_kl,INBDG1,Badagara
port_code_inkal1,base.state_in_kl,INKAL1,Kallai
port_code_inksg1,base.state_in_kl,INKSG1,Kasargod
port_code_inkvl1,base.state_in_kl,INKVL1,Kovalam
port_code_inmhe1,base.state_in_kl,INMHE1,Mahe
port_code_inpnn1,base.state_in_kl,INPNN1,Ponnani
port_code_incct6,base.state_in_kl,INCCT6,KINFRAFP SEZ / Kozhikkode
port_code_intvc6,base.state_in_kl,INTVC6,KINFRAA SEZ / Thiruvananthapuram
port_code_inerv6,base.state_in_kl,INERV6,Vallarpadom SEZ / Ernakulam
port_code_inerp6,base.state_in_kl,INERP6,Puthuvypeen SEZ / Ernakulam
port_code_inrkg1,base.state_in_kl,INRKG1,Rajakkamangalam
port_code_inmlp1,base.state_in_kl,INMLP1,Mallipuram
port_code_inlpr1,base.state_in_kl,INLPR1,Leapuram
port_code_inkdp1,base.state_in_kl,INKDP1,Kondiapetnam
port_code_inknd1,base.state_in_kl,INKND1,Kankudy
port_code_incnn1,base.state_in_kl,INCNN1,Cannanore
port_code_incok1,base.state_in_kl,INCOK1,Cochin Sea
port_code_inalf1,base.state_in_kl,INALF1,Allepey
port_code_incok6,base.state_in_kl,INCOK6,Cochin (EPZ/SEZ)
port_code_inarr6,base.state_in_kl,INARR6,Aroor
port_code_inccj4,base.state_in_kl,INCCJ4,Karipur(Calicut)
port_code_inmci1,base.state_in_kl,INMCI1,Minicoy Island
port_code_intel1,base.state_in_kl,INTEL1,Tellichery
port_code_invzj1,base.state_in_kl,INVZJ1,Vazhinjam
port_code_incok4,base.state_in_kl,INCOK4,Cochin
port_code_innee1,base.state_in_kl,INNEE1,Neendakara
port_code_inccj1,base.state_in_kl,INCCJ1,C.H. Kozikode
port_code_inazk1,base.state_in_kl,INAZK1,Azhikkal
port_code_inkym6,base.state_in_kl,INKYM6,Kottayam (10.11.09)
port_code_intcr6,base.state_in_kl,INTCR6,Thrissur ICD
port_code_inagi1,base.state_in_ld,INAGI1,Agatti Island
port_code_inkti1,base.state_in_ld,INKTI1,Kiltan Island
port_code_inadi1,base.state_in_ld,INADI1,Androth Island
port_code_inbtr1,base.state_in_ld,INBTR1,Bitra Island
port_code_incti1,base.state_in_ld,INCTI1,Chettat Island
port_code_inkvt1,base.state_in_ld,INKVT1,Kavaratti Island
port_code_inkdi1,base.state_in_ld,INKDI1,Kadmat lsland
port_code_inami1,base.state_in_ld,INAMI1,Amini IsTand
port_code_inmpr6,base.state_in_mp,INMPR6,Malanpur ICD
port_code_inkhd6,base.state_in_mp,INKHD6,Kheda -Dhar ICD
port_code_indha6,base.state_in_mp,INDHA6,Indore-Dhannad
port_code_inmdd6,base.state_in_mp,INMDD6,Mandideep
port_code_inidr4,base.state_in_mp,INIDR4,Indore
port_code_ingwl6,base.state_in_mp,INGWL6,Malanpur(Gwalior)
port_code_inind6,base.state_in_mp,ININD6,Pithampur
port_code_inidr6,base.state_in_mp,INIDR6,Indore (EPZ/SEZ)
port_code_inrtm6,base.state_in_mp,INRTM6,RATLAM(CONCOR)
port_code_ininb6,base.state_in_mp,ININB6,MPAKVN SEZ/Indore
port_code_inini6,base.state_in_mp,ININI6,IIPL SEZ / Indore
port_code_ininn6,base.state_in_mp,ININN6,Infosys SEZ / Indore
port_code_inint6,base.state_in_mp,ININT6,TCS SEZ / Indore
port_code_inach1,base.state_in_mh,INACH1,Achra
port_code_inbom4,base.state_in_mh,INBOM4,Bombay Air Cargo
port_code_innsa1,base.state_in_mh,INNSA1,Nhava Sheva Sea
port_code_invyd1,base.state_in_mh,INVYD1,Vijaydurg
port_code_invng1,base.state_in_mh,INVNG1,Vengurla
port_code_invsv1,base.state_in_mh,INVSV1,Varsava
port_code_invrd1,base.state_in_mh,INVRD1,Varavda
port_code_inutn1,base.state_in_mh,INUTN1,Uttan
port_code_inulw1,base.state_in_mh,INULW1,Ulwa
port_code_intmp1,base.state_in_mh,INTMP1,Trombay
port_code_intna1,base.state_in_mh,INTNA1,Thana
port_code_inthl1,base.state_in_mh,INTHL1,Thal
port_code_intrp1,base.state_in_mh,INTRP1,Tarapur
port_code_indeg1,base.state_in_mh,INDEG1,Deogad
port_code_indlb6,base.state_in_mh,INDLB6,Daulatabad ICD
port_code_indtw1,base.state_in_mh,INDTW1,Dantiwara
port_code_indhn1,base.state_in_mh,INDHN1,Dahanu
port_code_inbry1,base.state_in_mh,INBRY1,Borya
port_code_inbrm1,base.state_in_mh,INBRM1,Borlai – Mandla
port_code_inbsl6,base.state_in_mh,INBSL6,Bhusaval ICD
port_code_inbwn1,base.state_in_mh,INBWN1,Bhiwandi
port_code_inblp1,base.state_in_mh,INBLP1,Belapur
port_code_inkrp1,base.state_in_mh,INKRP1,Kiranpani
port_code_inkiw1,base.state_in_mh,INKIW1,Kelwa
port_code_inksh1,base.state_in_mh,INKSH1,Kelshi
port_code_inkrn1,base.state_in_mh,INKRN1,Karanja
port_code_inkly1,base.state_in_mh,INKLY1,Kalyan
port_code_injtp1,base.state_in_mh,INJTP1,Jaitapur
port_code_injgd1,base.state_in_mh,INJGD1,Jaigad
port_code_inhrn1,base.state_in_mh,INHRN1,Harnai
port_code_indig1,base.state_in_mh,INDIG1,Dighi Port
port_code_injnr4,base.state_in_mh,INJNR4,Nashik-Janori ACC
port_code_injnr6,base.state_in_mh,INJNR6,Nashik-Janori ICD
port_code_intlg6,base.state_in_mh,INTLG6,Pune-Talegoan ICD
port_code_inmwa6,base.state_in_mh,INMWA6,ICD Maliwada
port_code_inpvl6,base.state_in_mh,INPVL6,Panvel
port_code_inswd1,base.state_in_mh,INSWD1,Shriwardhan
port_code_instp1,base.state_in_mh,INSTP1,Satpati
port_code_inrjr1,base.state_in_mh,INRJR1,Rajpuri
port_code_inprg1,base.state_in_mh,INPRG1,Purangad
port_code_inpsh1,base.state_in_mh,INPSH1,Palshet
port_code_innvt1,base.state_in_mh,INNVT1,Nivti
port_code_innwp1,base.state_in_mh,INNWP1,Newapur
port_code_inndg1,base.state_in_mh,INNDG1,Nandgaon
port_code_inmrd1,base.state_in_mh,INMRD1,Murad
port_code_ingrd6,base.state_in_mh,INGRD6,Mumbai DP-II
port_code_ingrr6,base.state_in_mh,INGRR6,Mumbai -DP-I
port_code_inmra1,base.state_in_mh,INMRA1,Mora
port_code_inmrj6,base.state_in_mh,INMRJ6,Miraj
port_code_inmnr1,base.state_in_mh,INMNR1,Manori
port_code_inmnw1,base.state_in_mh,INMNW1,Mandwa
port_code_inmlw1,base.state_in_mh,INMLW1,Malwan
port_code_inkmb1,base.state_in_mh,INKMB1,Kumbharu
port_code_inbsn1,base.state_in_mh,INBSN1,Bassein
port_code_inbkt1,base.state_in_mh,INBKT1,Bankot
port_code_inbnd1,base.state_in_mh,INBND1,Bandra
port_code_inanl1,base.state_in_mh,INANL1,Arnala
port_code_inabg1,base.state_in_mh,INABG1,Alibag
port_code_inbom1,base.state_in_mh,INBOM1,Bombay Sea
port_code_indhp1,base.state_in_mh,INDHP1,Dabhol Port
port_code_inred1,base.state_in_mh,INRED1,Redi
port_code_inkhp6,base.state_in_mh,INKHP6,Khopta (EPZ/SEZ)
port_code_inbom6,base.state_in_mh,INBOM6,Mumbai (EPZ/SEZ)
port_code_incch6,base.state_in_mh,INCCH6,Chinchwad (ICD)
port_code_inwal6,base.state_in_mh,INWAL6,Waluj(Aurangabad)
port_code_inpmp6,base.state_in_mh,INPMP6,Pimpri
port_code_innsk6,base.state_in_mh,INNSK6,Nasik
port_code_inngp6,base.state_in_mh,INNGP6,Nagpur
port_code_inmul6,base.state_in_mh,INMUL6,Mulund
port_code_injal6,base.state_in_mh,INJAL6,Jalgaon
port_code_indig6,base.state_in_mh,INDIG6,Dighi(Pune)
port_code_innag4,base.state_in_mh,INNAG4,Nagpur
port_code_inpnq4,base.state_in_mh,INPNQ4,Pune
port_code_indhu1,base.state_in_mh,INDHU1,Dahanu
port_code_inrvd1,base.state_in_mh,INRVD1,Revdanda
port_code_inrtc1,base.state_in_mh,INRTC1,Ratnagiri
port_code_inrnr1,base.state_in_mh,INRNR1,Ranpar
port_code_inpnv6,base.state_in_mh,INPNV6,ARSHIYA-SEZ/PANVEL
port_code_indmt1,base.state_in_mh,INDMT1,Dharamtar
port_code_inbap6,base.state_in_mh,INBAP6,MultiService-SEZ
port_code_inklm6,base.state_in_mh,INKLM6,Multi Service-SEZ Kalambolli
port_code_inbau6,base.state_in_mh,INBAU6,IT/ITES-A-SEZ/Ulwe
port_code_inbai6,base.state_in_mh,INBAI6,IT/ITES-B-SEZ/Ulwe
port_code_inbag6,base.state_in_mh,INBAG6,Gem & Jewellery-SEZ/Ulwe
port_code_inbam6,base.state_in_mh,INBAM6,Multi Service-SEZ/Ulwe(Distt. Raigad)
port_code_inbat6,base.state_in_mh,INBAT6,IT/ITES-C SEZ/Ulwe
port_code_ainpnq6,base.state_in_mh,AINPNQ 6,Hadapsar, SEZ, Pune
port_code_inmuc6,base.state_in_mh,INMUC6,SUNSTREAM CITY PVT.
port_code_inpsi6,base.state_in_mh,INPSI6,SYNTEL INTERNATIONAL
port_code_inpmt6,base.state_in_mh,INPMT6,MAGARPATTA TOWNSHIP
port_code_inpit6,base.state_in_mh,INPIT6,INFOSYS TECHNOLOGIES
port_code_inpek6,base.state_in_mh,INPEK6,EON KHARADI INFRASTRUCTURE
port_code_inkrm6,base.state_in_mh,INKRM6,Maharashtra Airport
port_code_inair6,base.state_in_mh,INAIR6,Serene Properties Private Limited
port_code_invkh6,base.state_in_mh,INVKH6,HNB SEZ/ MUMBAI
port_code_inchj6,base.state_in_mh,INCHJ6,WWIL ICD/ WARDHA
port_code_indpc4,base.state_in_mh,INDPC4,PCCCC, Bandra- Kurla Complex
port_code_inbng6,base.state_in_mh,INBNG6,MMAuHmAbGaAiONICD/THAN
port_code_inaig6,base.state_in_mh,INAIG6,GEPL SEZ/Thane
port_code_inawm6,base.state_in_mh,INAWM6,MIDC SEZ/AURANGABAD
port_code_indid6,base.state_in_mh,INDID6,MIDC SEZ/NANDED
port_code_inpum6,base.state_in_mh,INPUM6,MIDC SEZ/PUNE
port_code_inkle6,base.state_in_mh,INKLE6,MIDC SEZ / RAIGAD
port_code_instu6,base.state_in_mh,INSTU6,MIDC SEZ / KESURDE
port_code_innki6,base.state_in_mh,INNKI6,IIIL SEZ / SINNAR
port_code_instm6,base.state_in_mh,INSTM6,MIDC PHALTAN SEZ / SATARA
port_code_inpnu6,base.state_in_mh,INPNU6,MSFPL SEZ / PUNE
port_code_inaww6,base.state_in_mh,INAWW6,WIDL SEZ / AURANGABAD
port_code_inwrr6,base.state_in_mh,INWRR6,WPCL CHANDRAPUR
port_code_inccw6,base.state_in_mh,INCCW6,WIPRO / PUNE
port_code_intgn6,base.state_in_mh,INTGN6,KEIPL / PUNE
port_code_inpun6,base.state_in_mh,INPUN6,KBTV / PUNE
port_code_inpne6,base.state_in_mh,INPNE6,NTPL / PUNE
port_code_inccq6,base.state_in_mh,INCCQ6,QBP / PUNE
port_code_inmreb,base.state_in_mn,INMREB,Moreh
port_code_inimf4,base.state_in_mn,INIMF4,Imphal
port_code_inbgmb,base.state_in_ml,INBGMB,Baghmara
port_code_inbrab,base.state_in_ml,INBRAB,Barsora
port_code_insbzb,base.state_in_ml,INSBZB,Shella Bazar
port_code_inrgub,base.state_in_ml,INRGUB,Ryngku
port_code_inmghb,base.state_in_ml,INMGHB,Mahendraganj
port_code_inghpb,base.state_in_ml,INGHPB,Ghasuapara
port_code_indwkb,base.state_in_ml,INDWKB,Dawki
port_code_indlub,base.state_in_ml,INDLUB,Dalu
port_code_inbolb,base.state_in_ml,INBOLB,Bolanganj
port_code_inbltb,base.state_in_ml,INBLTB,Balet
port_code_inchpb,base.state_in_mz,INCHPB,Champai
port_code_indmrb,base.state_in_nl,INDMRB,Demagir
port_code_inbbp1,base.state_in_or,INBBP1,Bahabal Pur
port_code_inprt1,base.state_in_or,INPRT1,Paradeep
port_code_inbbi4,base.state_in_or,INBBI4,Bhubaneswar
port_code_ingpr1,base.state_in_or,INGPR1,Gopalpur
port_code_inskd6,base.state_in_or,INSKD6,Kalinganagar
port_code_inbbs6,base.state_in_or,INBBS6,OIIDC SEZ/Bhubneshwar
port_code_incas6,base.state_in_or,INCAS6,SAPL SEZ / Ganjam
port_code_inkrk1,base.state_in_py,INKRK1,Karaikal
port_code_inpny1,base.state_in_py,INPNY1,Pondicherry
port_code_inpny6,base.state_in_py,INPNY6,Pondicherry ICD
port_code_inlud6,base.state_in_pb,INLUD6,Ludhiana
port_code_inasr2,base.state_in_pb,INASR2,Amritsar Railway Station
port_code_inatt2,base.state_in_pb,INATT2,Attari Railway Station
port_code_inatrb,base.state_in_pb,INATRB,Attari Road
port_code_indpr6,base.state_in_pb,INDPR6,DAPPER
port_code_inatq4,base.state_in_pb,INATQ4,Rajasansi(Amritsar)
port_code_injuuc6,base.state_in_pb,INJUC6,Jalandhar
port_code_inldh6,base.state_in_pb,INLDH6,Ludhiana
port_code_inasr6,base.state_in_pb,INASR6,Amritsar
port_code_inbti6,base.state_in_pb,INBTI6,Bhatinda
port_code_injai6,base.state_in_rj,INJAI6,Jaipur ICD
port_code_intha6,base.state_in_rj,INTHA6,Thar Dry Port – Jodhpur ICD
port_code_inmnb2,base.state_in_rj,INMNB2,Munabao Railway Station
port_code_inkku6,base.state_in_rj,INKKU6,Kanakpura – Jaipur ICD
port_code_inbrn6,base.state_in_rj,INBRN6,Jodhpur- Boranda (EPZ/SEZ)
port_code_injai5,base.state_in_rj,INJAI5,Jaipur
port_code_injsz6,base.state_in_rj,INJSZ6,Jaipur – Sitapur (EPZ/SEZ)
port_code_inbgk6,base.state_in_rj,INBGK6,Bhagat ki Kothi – Jodhpur ICD
port_code_inbmr2,base.state_in_rj,INBMR2,Barmer Railway Station
port_code_injai4,base.state_in_rj,INJAI4,Jaipur
port_code_inbhl6,base.state_in_rj,INBHL6,Bhilwara
port_code_injux6,base.state_in_rj,INJUX6,Jodhpur
port_code_inudz6,base.state_in_rj,INUDZ6,Udaipur
port_code_inktt6,base.state_in_rj,INKTT6,Kota
port_code_inbwd6,base.state_in_rj,INBWD6,Bhiwadi
port_code_inchmb,base.state_in_sk,INCHMB,Chamurchi
port_code_inajj6,base.state_in_tn,INAJJ6,Arakkonam – Melpakkam – Chennai
port_code_inixm6,base.state_in_tn,INIXM6,Madurai ICD
port_code_inche6,base.state_in_tn,INCHE6,Tiruppur – Chettipalayam CFS
port_code_inkar6,base.state_in_tn,INKAR6,KARUR
port_code_inigu6,base.state_in_tn,INIGU6,Coimbatore – Irugur ICD
port_code_inmaa6,base.state_in_tn,INMAA6,Chennai (EPZ/SEZ)
port_code_inmaa4,base.state_in_tn,INMAA4,Chennai Air Cargo
port_code_intut1,base.state_in_tn,INTUT1,Tuticorin Sea
port_code_intut6,base.state_in_tn,INTUT6,Tuticorin ICD
port_code_inmaa1,base.state_in_tn,INMAA1,Chennai Sea
port_code_indsk1,base.state_in_tn,INDSK1,Dhanu – Shkodi
port_code_inksp1,base.state_in_tn,INKSP1,Kulasekarapatnam
port_code_inptn1,base.state_in_tn,INPTN1,Portonovo
port_code_intph1,base.state_in_tn,INTPH1,Thopputhurai
port_code_intde6,base.state_in_tn,INTDE6,Tudiyalur – Coimbatore ICD
port_code_intnd1,base.state_in_tn,INTND1,Tondi
port_code_intho6,base.state_in_tn,INTHO6,Tiruppur – Thottiplayam ICD
port_code_inrwr1,base.state_in_tn,INRWR1,Rameshwaram
port_code_inpmb1,base.state_in_tn,INPMB1,Pamban
port_code_inkkr1,base.state_in_tn,INKKR1,Kilakari
port_code_inchl1,base.state_in_tn,INCHL1,Colachel
port_code_intrl6,base.state_in_tn,INTRL6,Tiruvallur ICD
port_code_inilp6,base.state_in_tn,INILP6,Irungattukottai-ILP ICD
port_code_incdl1,base.state_in_tn,INCDL1,Cuddalore
port_code_innpt1,base.state_in_tn,INNPT1,Nagapattinam
port_code_intyr1,base.state_in_tn,INTYR1,Tirukkadayyur
port_code_invkm1,base.state_in_tn,INVKM1,Valinokkam
port_code_insll6,base.state_in_tn,INSLL6,Singnallur
port_code_intup6,base.state_in_tn,INTUP6,Tirupur
port_code_insxt6,base.state_in_tn,INSXT6,Salem
port_code_incbe6,base.state_in_tn,INCBE6,Coimbatore
port_code_intrz4,base.state_in_tn,INTRZ4,Tiruchirapalli
port_code_incjb4,base.state_in_tn,INCJB4,Coimbatore
port_code_invep1,base.state_in_tn,INVEP1,Veppalodai
port_code_inram1,base.state_in_tn,INRAM1,Rameshwaram
port_code_inmdp1,base.state_in_tn,INMDP1,Mandapam
port_code_incgi6,base.state_in_tn,INCGI6,MWCDL-IT SEZ/Chengalpattu
port_code_incga6,base.state_in_tn,INCGA6,MWCDL-Apparel-sez/Chengalpattu
port_code_incgl6,base.state_in_tn,INCGL6,MWCDL-Auto ANCILLARIES
port_code_incjn6,base.state_in_tn,INCJN6,NIPL-SEZ Sriperumbudur
port_code_incjs6,base.state_in_tn,INCJS6,SIPCOT Hi-Tech-SEZ Sriperumbudur
port_code_incjo6,base.state_in_tn,INCJO6,SIPCOT
port_code_incjf6,base.state_in_tn,INCJF6,FTIL-SEZ Sriperumbudur
port_code_incbs6,base.state_in_tn,INCBS6,SE&C Ltd-SEZ/Coimbatore
port_code_invtc6,base.state_in_tn,INVTC6,CHEYYAR-SEZ/Vellore
port_code_inten6,base.state_in_tn,INTEN6,SIPCOT-Gangaikondan-SEZ/Tirunelveli
port_code_intnc6,base.state_in_tn,INTNC6,CCCL Infrastructure Ltd.
port_code_innnn6,base.state_in_tn,INNNN6,AMRL International Tech City Ltd.
port_code_intbm6,base.state_in_tn,INTBM6,SPEHZP/L -NSaEnZg/Kuannerciheepuram
port_code_inukl6,base.state_in_tn,INUKL6,ETLISL-SEZ/Erode
port_code_inpys6,base.state_in_tn,INPYS6,SIPCOT-SEZ/Erode
port_code_ingdp6,base.state_in_tn,INGDP6,FLLPL-SEZ/Thirruvallur
port_code_inenr1,base.state_in_tn,INENR1,Ennore
port_code_intvt6,base.state_in_tn,INTVT6,ICD/Tondiarpet Chennai
port_code_inkat1,base.state_in_tn,INKAT1,KATTUPALLI
port_code_intbt6,base.state_in_tn,INTBT6,TCSL SEZ / SIRUSERI
port_code_incsp6,base.state_in_tn,INCSP6,SIPL SEZ / KANCHEEPURAM
port_code_incji6,base.state_in_tn,INCJI6,IG3I SEZ / KANCHEEPURAM
port_code_incjd6,base.state_in_tn,INCJD6,DLFC SEZ / KANCHEEPURAM
port_code_incec6,base.state_in_tn,INCEC6,ECTN SEZ / COIMBATORE
port_code_incje6,base.state_in_tn,INCJE6,ECTN SEZ / KANCHEEPURAM
port_code_incja6,base.state_in_tn,INCJA6,AEIP SEZ / KANCHEEPURAM
port_code_incsv6,base.state_in_tn,INCSV6,SVPL SEZ / COIMBATORE
port_code_incge6,base.state_in_tn,INCGE6,ETA SEZ / KANCHEEPURAM
port_code_incnc6,base.state_in_tn,INCNC6,NCTL SEZ / KANCHEEPURAM
port_code_intbc6,base.state_in_tn,INTBC6,CTSI SEZ / SIRUSERI
port_code_inmas6,base.state_in_tn,INMAS6,TIPL SEZ / CHENNAI
port_code_intlt6,base.state_in_tn,INTLT6,LTSL SEZ / TIRUVALLUR
port_code_intbh6,base.state_in_tn,INTBH6,HIRL SEZ / KANCHEEPURAM
port_code_incbt6,base.state_in_tn,INCBT6 ,TDPL SEZ / COIMBATORE
port_code_invlr6,base.state_in_tn,INVLR6,SIPC SEZ / VELLORE
port_code_incjv6,base.state_in_tn,INCJV6,VTPL SEZ / KANCHEEPURAM
port_code_inmec6,base.state_in_tn,INMEC6,ECTN SEZ / MADURAI – I
port_code_inmdc6,base.state_in_tn,INMDC6,ECTN SEZ / MADURAI – II
port_code_insxe6,base.state_in_tn,INSXE6,ECTN SEZ / SALEM
port_code_inmdr6,base.state_in_tn,INMDR6,RTPL SEZ / MADURAI
port_code_incfi6,base.state_in_tn,INCFI6,FIPL SEZ / KANCHEEPURAM
port_code_inhsi6,base.state_in_tn,INHSI6,SIPC SEZ / KRISHNAGIRI
port_code_inmae6,base.state_in_tn,INMAE6,ECTNL SEZ / Gangaikondan
port_code_intbs6,base.state_in_tn,INTBS6,HTL SEZ / Sireseri
port_code_inagtb,base.state_in_tr,INAGTB,Agartala
port_code_inmhgb,base.state_in_tr,INMHGB,Mahurighat
port_code_insabb,base.state_in_tr,INSABB,Sabroom
port_code_insmpb,base.state_in_tr,INSMPB,Srimantapur
port_code_inrgbb,base.state_in_tr,INRGBB,Old Raghna Bazar
port_code_inkwgb,base.state_in_tr,INKWGB,Khowaighat
port_code_indhlb,base.state_in_tr,INDHLB,Dhalaighat
port_code_inkelb,base.state_in_tr,INKELB,Kel Sahar Subdivision
port_code_inbsab,base.state_in_up,INBSAB,Banbasa
port_code_ingaib,base.state_in_up,INGAIB,Gauriphanta
port_code_inrdt6,base.state_in_up,INRDT6,Kota-Ravtha Road
port_code_ingjib,base.state_in_up,INGJIB,Gunji
port_code_injhob,base.state_in_up,INJHOB,Jhulaghat (Pithoragarh)
port_code_inktgb,base.state_in_up,INKTGB,Katarniyaghat
port_code_inngrb,base.state_in_up,INNGRB,Nepalgunj Road
port_code_inmbs6,base.state_in_up,INMBS6,Madhosingh ICD
port_code_inlon6,base.state_in_up,INLON6,ICD Loni
port_code_incpl6,base.state_in_up,INCPL6,Dadri-CGML
port_code_inbdh6,base.state_in_up,INBDH6,ICD Badohi
port_code_inttp6,base.state_in_up,INTTP6,Dadri TTPL
port_code_inapl6,base.state_in_up,INAPL6,Dadri-ACPL CFS
port_code_intknb,base.state_in_up,INTKNB,Tikonia
port_code_insnlb,base.state_in_up,INSNLB,Sonauli
port_code_inkwab,base.state_in_up,INKWAB,Khunwa
port_code_inknu6,base.state_in_up,INKNU6,Kanpur – JRY (ICD)
port_code_injwab,base.state_in_up,INJWAB,Jarwa
port_code_insjr6,base.state_in_up,INSJR6,Greater Noida-Surajpur
port_code_indlab,base.state_in_up,INDLAB,Dharchula
port_code_inbnyb,base.state_in_up,INBNYB,Berhni
port_code_instt6,base.state_in_up,INSTT6,Dadri – STTPL (CFS)
port_code_inagr4,base.state_in_up,INAGR4,Agra
port_code_inder6,base.state_in_up,INDER6,Noida-Dadri (ICD)
port_code_inmbc6,base.state_in_up,INMBC6,Moradabad (EPZ/SEZ)
port_code_innda6,base.state_in_up,INNDA6,Noida (EPZ/SEZ)
port_code_insre6,base.state_in_up,INSRE6,Saharanpur
port_code_inmtc6,base.state_in_up,INMTC6,Meerut
port_code_inmbd6,base.state_in_up,INMBD6,Pakwara (Moradabad)
port_code_incpc6,base.state_in_up,INCPC6,Kanpur
port_code_inbsb6,base.state_in_up,INBSB6,Varanasi
port_code_inblj6,base.state_in_up,INBLJ6,Agra
port_code_invns4,base.state_in_up,INVNS4,Varanasi
port_code_inpnk6,base.state_in_up,INPNK6,KLPPL-ICD/PANKI
port_code_inbek4,base.state_in_up,INBEK4,Bareilly
port_code_inlko4,base.state_in_up,INLKO4,Lucknow
port_code_inknu4,base.state_in_up,INKNU4,Kanpur
port_code_inbul6,base.state_in_up,INBUL6,AN FTWZ LTD – FTWZ/ BULANDSHAHR
port_code_inalp6,base.state_in_up,INALP6,Dadri, Greater Noida
port_code_innoi6,base.state_in_up,INNOI6,LONI-1 ICD Ghaziabad
port_code_inccu4,base.state_in_wb,INCCU4,Kolkata Air Cargo
port_code_ingtzb,base.state_in_wb,INGTZB,Getandah
port_code_inhlib,base.state_in_wb,INHLIB,Hilli
port_code_injigb,base.state_in_wb,INJIGB,Jaigaon
port_code_insng2,base.state_in_wb,INSNG2,Singabad Railway Station
port_code_inrng2,base.state_in_wb,INRNG2,Ranaghat Railway Station
port_code_inrdp2,base.state_in_wb,INRDP2,Radhikapur Railway Station
port_code_inptpb,base.state_in_wb,INPTPB,Petrapole Road
port_code_inpntb,base.state_in_wb,INPNTB,Pan itanki (Naxabari)
port_code_innknb,base.state_in_wb,INNKNB,Namkhana
port_code_inlglb,base.state_in_wb,INLGLB,Lalgola Town
port_code_inmhdb,base.state_in_wb,INMHDB,Kotawalighat (Mohedipur)
port_code_injpgb,base.state_in_wb,INJPGB,Jalpaiguri
port_code_indur6,base.state_in_wb,INDUR6,ICD Durgapur
port_code_intngb,base.state_in_wb,INTNGB,Tungi
port_code_inttsb,base.state_in_wb,INTTSB,T.T. Shed (Kidcerpore)
port_code_inskpb,base.state_in_wb,INSKPB,Sukhia Pokhari
port_code_instib,base.state_in_wb,INSTIB,Sitai
port_code_inhglb,base.state_in_wb,INHGLB,Hingalganj
port_code_ingjxb,base.state_in_wb,INGJXB,Ghojadanga
port_code_inged2,base.state_in_wb,INGED2,Gede Railway Station
port_code_inptp8,base.state_in_wb,INPTP8,Patrapole
port_code_incbdb,base.state_in_wb,INCBDB,Changrabandh
port_code_infbrb,base.state_in_wb,INFBRB,Fulbari
port_code_inccu1,base.state_in_wb,INCCU1,Kolkata Sea
port_code_inhal1,base.state_in_wb,INHAL1,Haldia
port_code_inslt6,base.state_in_wb,INSLT6,Salt Lake (EPZ/SEZ)
port_code_inflt6,base.state_in_wb,INFLT6,Falta (EPZ/SEZ)
port_code_inixb4,base.state_in_wb,INIXB4,Bagdogra
port_code_inbnt6,base.state_in_wb,INBNT6,TCS SEZ/Kolkata
port_code_inbxr6,base.state_in_wb,INBXR6,DLF SEZ/Kolkata
port_code_inbnx6,base.state_in_wb,INBNX6,UNITECH SEZ/Kolkata
port_code_inbnk6,base.state_in_wb,INBNK6,Kolkata IT Park/Bantala
port_code_inbnw6,base.state_in_wb,INBNW6,Wipro SEZ/ Kolkata

```

## File: data\l10n_in_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <menuitem id="account_reports_in_statements_menu" name="India" parent="account.menu_finance_reports" sequence="0"/>

        <record id="indian_chart_template_standard" model="account.chart.template">
            <field name="name">Indian Chart of Accounts - Standard</field>
            <field name="bank_account_code_prefix">1002</field>
            <field name="cash_account_code_prefix">1001</field>
            <field name="transfer_account_code_prefix">1008</field>
            <field name="code_digits">6</field>
            <field name="currency_id" ref="base.INR"/>
        </record>

        <record id="sgst_tag_account" model="account.account.tag">
            <field name="name">SGST</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="cgst_tag_account" model="account.account.tag">
            <field name="name">CGST</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="igst_tag_account" model="account.account.tag">
            <field name="name">IGST</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="cess_tag_account" model="account.account.tag">
            <field name="name">CESS</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="p10055" model="account.account.template">
            <field name="name">CESS Receivable</field>
            <field name="code">10055</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets"/>
            <field name="reconcile" eval="False"/>
            <field name="chart_template_id" ref="indian_chart_template_standard"/>
            <field name="tag_ids" eval="[(6,0,[ref('cess_tag_account'),])]"/>
        </record>
        <record id="p10056" model="account.account.template">
            <field name="name">Tax Receivable</field>
            <field name="code">10056</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets"/>
            <field name="reconcile" eval="False"/>
            <field name="chart_template_id" ref="indian_chart_template_standard"/>
        </record>
        <record model="account.account.template" id="p11235">
            <field name="name">CESS Payable</field>
            <field name="code">11235</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities"/>
            <field name="reconcile" eval="False"/>
            <field name="chart_template_id" ref="indian_chart_template_standard"/>
            <field name="tag_ids" eval="[(6,0,[ref('cess_tag_account'),])]"/>
        </record>
        <record id="p11236" model="account.account.template">
            <field name="name">Tax Payable</field>
            <field name="code">11236</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities"/>
            <field name="reconcile" eval="False"/>
            <field name="chart_template_id" ref="indian_chart_template_standard"/>
        </record>
</odoo>

```

## File: data\l10n_in_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="indian_chart_template_standard" model="account.chart.template">
        <field name="property_account_receivable_id" ref="p10040"/>
        <field name="property_account_payable_id" ref="p11211"/>
        <field name="property_account_expense_categ_id" ref="p2107"/>
        <field name="property_account_income_categ_id" ref="p20011"/>
        <field name="income_currency_exchange_account_id" ref="p2013"/>
        <field name="expense_currency_exchange_account_id" ref="p2117"/>
        <field name="default_pos_receivable_account_id" ref="p10041" />
    </record>
</odoo>

```

## File: data\res_country_state_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="state_in_ot" model="res.country.state">
        <field name="name">Other Territory</field>
        <field name="code">IN_OT</field>
        <field name="country_id" ref="base.in"/>
        <field name="l10n_in_tin">97</field>
    </record>

    <record id="base.state_in_an" model="res.country.state">
        <field name="l10n_in_tin">35</field>
    </record>
    <record id="base.state_in_ap" model="res.country.state">
        <field name="l10n_in_tin">37</field>
    </record>
    <record id="base.state_in_ar" model="res.country.state">
        <field name="l10n_in_tin">12</field>
    </record>
    <record id="base.state_in_as" model="res.country.state">
        <field name="l10n_in_tin">18</field>
    </record>
    <record id="base.state_in_br" model="res.country.state">
        <field name="l10n_in_tin">10</field>
    </record>
    <record id="base.state_in_ch" model="res.country.state">
        <field name="l10n_in_tin">04</field>
    </record>
    <record id="base.state_in_cg" model="res.country.state">
        <field name="l10n_in_tin">22</field>
    </record>
    <record id="base.state_in_dn" model="res.country.state">
        <field name="l10n_in_tin">26</field>
    </record>
    <record id="base.state_in_dd" model="res.country.state">
        <field name="l10n_in_tin">25</field>
    </record>
    <record id="base.state_in_dl" model="res.country.state">
        <field name="l10n_in_tin">07</field>
    </record>
    <record id="base.state_in_ga" model="res.country.state">
        <field name="l10n_in_tin">30</field>
    </record>
    <record id="base.state_in_gj" model="res.country.state">
        <field name="l10n_in_tin">24</field>
    </record>
    <record id="base.state_in_hr" model="res.country.state">
        <field name="l10n_in_tin">06</field>
    </record>
    <record id="base.state_in_hp" model="res.country.state">
        <field name="l10n_in_tin">02</field>
    </record>
    <record id="base.state_in_jk" model="res.country.state">
        <field name="l10n_in_tin">01</field>
    </record>
    <record id="base.state_in_jh" model="res.country.state">
        <field name="l10n_in_tin">20</field>
    </record>
    <record id="base.state_in_ka" model="res.country.state">
        <field name="l10n_in_tin">29</field>
    </record>
    <record id="base.state_in_kl" model="res.country.state">
        <field name="l10n_in_tin">32</field>
    </record>
    <record id="base.state_in_ld" model="res.country.state">
        <field name="l10n_in_tin">31</field>
    </record>
    <record id="base.state_in_mp" model="res.country.state">
        <field name="l10n_in_tin">23</field>
    </record>
    <record id="base.state_in_mh" model="res.country.state">
        <field name="l10n_in_tin">27</field>
    </record>
    <record id="base.state_in_mn" model="res.country.state">
        <field name="l10n_in_tin">14</field>
    </record>
    <record id="base.state_in_ml" model="res.country.state">
        <field name="l10n_in_tin">17</field>
    </record>
    <record id="base.state_in_mz" model="res.country.state">
        <field name="l10n_in_tin">15</field>
    </record>
    <record id="base.state_in_nl" model="res.country.state">
        <field name="l10n_in_tin">13</field>
    </record>
    <record id="base.state_in_or" model="res.country.state">
        <field name="l10n_in_tin">21</field>
    </record>
    <record id="base.state_in_py" model="res.country.state">
        <field name="l10n_in_tin">34</field>
    </record>
    <record id="base.state_in_pb" model="res.country.state">
        <field name="l10n_in_tin">03</field>
    </record>
    <record id="base.state_in_rj" model="res.country.state">
        <field name="l10n_in_tin">08</field>
    </record>
    <record id="base.state_in_sk" model="res.country.state">
        <field name="l10n_in_tin">11</field>
    </record>
    <record id="base.state_in_tn" model="res.country.state">
        <field name="l10n_in_tin">33</field>
    </record>
    <record id="base.state_in_ts" model="res.country.state">
        <field name="l10n_in_tin">36</field>
    </record>
    <record id="base.state_in_tr" model="res.country.state">
        <field name="l10n_in_tin">16</field>
    </record>
    <record id="base.state_in_up" model="res.country.state">
        <field name="l10n_in_tin">09</field>
    </record>
    <record id="base.state_in_uk" model="res.country.state">
        <field name="l10n_in_tin">05</field>
    </record>
    <record id="base.state_in_wb" model="res.country.state">
        <field name="l10n_in_tin">19</field>
    </record>

</odoo>

```

## File: data\uom_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <!--
        l10n_in_code use in export GSTR hsn section report.
    -->
    <record id="uom.product_uom_unit" model="uom.uom">
        <field name="l10n_in_code">UNT-UNITS</field>
    </record>
    <record id="uom.product_uom_dozen" model="uom.uom">
        <field name="l10n_in_code">DOZ-DOZENS</field>
    </record>
    <record id="uom.product_uom_kgm" model="uom.uom">
        <field name="l10n_in_code">KGS-KILOGRAMS</field>
    </record>
    <record id="uom.product_uom_gram" model="uom.uom">
        <field name="l10n_in_code">GMS-GRAMMES</field>
    </record>
    <record id="uom.product_uom_day" model="uom.uom">
        <field name="l10n_in_code">OTH-OTHERS</field>
    </record>
    <record id="uom.product_uom_hour" model="uom.uom">
        <field name="l10n_in_code">OTH-OTHERS</field>
    </record>
    <record id="uom.product_uom_ton" model="uom.uom">
        <field name="l10n_in_code">TON-TONNES</field>
    </record>
    <record id="uom.product_uom_meter" model="uom.uom">
        <field name="l10n_in_code">MTR-METERS</field>
    </record>
    <record id="uom.product_uom_km" model="uom.uom">
        <field name="l10n_in_code">KME-KILOMETRE</field>
    </record>
    <record id="uom.product_uom_cm" model="uom.uom">
        <field name="l10n_in_code">CMS-CENTIMETERS</field>
    </record>
    <record id="uom.product_uom_litre" model="uom.uom">
        <field name="l10n_in_code">LTR-LITRES</field>
    </record>
    <record id="uom.product_uom_cubic_meter" model="uom.uom">
        <field name="l10n_in_code">CBM-CUBIC METERS</field>
    </record>
    <record id="uom.product_uom_lb" model="uom.uom">
        <field name="l10n_in_code">OTH-OTHERS</field>
    </record>
    <record id="uom.product_uom_oz" model="uom.uom">
        <field name="l10n_in_code">OTH-OTHERS</field>
    </record>
    <record id="uom.product_uom_inch" model="uom.uom">
        <field name="l10n_in_code">OTH-OTHERS</field>
    </record>
    <record id="uom.product_uom_foot" model="uom.uom">
        <field name="l10n_in_code">OTH-OTHERS</field>
    </record>
    <record id="uom.product_uom_mile" model="uom.uom">
        <field name="l10n_in_code">OTH-OTHERS</field>
    </record>
    <record id="uom.product_uom_floz" model="uom.uom">
        <field name="l10n_in_code">OTH-OTHERS</field>
    </record>
    <record id="uom.product_uom_qt" model="uom.uom">
        <field name="l10n_in_code">OTH-OTHERS</field>
    </record>
    <record id="uom.product_uom_gal" model="uom.uom">
        <field name="l10n_in_code">UGS-US GALLONS</field>
    </record>
    <record id="uom.product_uom_cubic_inch" model="uom.uom">
        <field name="l10n_in_code">OTH-OTHERS</field>
    </record>
    <record id="uom.product_uom_cubic_foot" model="uom.uom">
        <field name="l10n_in_code">OTH-OTHERS</field>
    </record>
</odoo>

```

## File: migrations\9.0.2.0\pre-set_tags_and_taxes_updatable.py

```python
# -*- coding: utf-8 -*-

import odoo

def migrate(cr, version):
    registry = odoo.registry(cr.dbname)
    from odoo.addons.account.models.chart_template import migrate_set_tags_and_taxes_updatable
    migrate_set_tags_and_taxes_updatable(cr, registry, 'l10n_in')

```

## File: models\account.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError
from odoo import tools


class AccountJournal(models.Model):
    _inherit = "account.journal"

    # Use for filter import and export type.
    l10n_in_gstin_partner_id = fields.Many2one('res.partner', string="GSTIN Unit", ondelete="restrict", help="GSTIN related to this journal. If empty then consider as company GSTIN.")

    def name_get(self):
        """
            Add GSTIN number in name as suffix so user can easily find the right journal.
            Used super to ensure nothing is missed.
        """
        result = super().name_get()
        result_dict = dict(result)
        indian_journals = self.filtered(lambda j: j.company_id.country_id.code == 'IN' and
            j.l10n_in_gstin_partner_id and j.l10n_in_gstin_partner_id.vat)
        for journal in indian_journals:
            name = result_dict[journal.id]
            name += "- %s" % (journal.l10n_in_gstin_partner_id.vat)
            result_dict[journal.id] = name
        return list(result_dict.items())


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    def init(self):
        tools.create_index(self._cr, 'account_move_line_move_product_index', self._table, ['move_id', 'product_id'])

    @api.depends('move_id.line_ids', 'move_id.line_ids.tax_line_id', 'move_id.line_ids.debit', 'move_id.line_ids.credit')
    def _compute_tax_base_amount(self):
        aml = self.filtered(lambda l: l.company_id.country_id.code == 'IN' and l.tax_line_id  and l.product_id)
        for move_line in aml:
            base_lines = move_line.move_id.line_ids.filtered(lambda line: move_line.tax_line_id in line.tax_ids and move_line.product_id == line.product_id)
            move_line.tax_base_amount = abs(sum(base_lines.mapped('balance')))
        remaining_aml = self - aml
        if remaining_aml:
            return super(AccountMoveLine, remaining_aml)._compute_tax_base_amount()


class AccountTax(models.Model):
    _inherit = 'account.tax'

    l10n_in_reverse_charge = fields.Boolean("Reverse charge", help="Tick this if this tax is reverse charge. Only for Indian accounting")

    def get_grouping_key(self, invoice_tax_val):
        """ Returns a string that will be used to group account.invoice.tax sharing the same properties"""
        key = super(AccountTax, self).get_grouping_key(invoice_tax_val)
        if self.company_id.country_id.code == 'IN':
            key += "-%s-%s"% (invoice_tax_val.get('l10n_in_product_id', False),
                invoice_tax_val.get('l10n_in_uom_id', False))
        return key

```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class AccountMove(models.Model):
    _inherit = "account.move"

    amount_total_words = fields.Char("Total (In Words)", compute="_compute_amount_total_words")
    l10n_in_gst_treatment = fields.Selection([
            ('regular', 'Registered Business - Regular'),
            ('composition', 'Registered Business - Composition'),
            ('unregistered', 'Unregistered Business'),
            ('consumer', 'Consumer'),
            ('overseas', 'Overseas'),
            ('special_economic_zone', 'Special Economic Zone'),
            ('deemed_export', 'Deemed Export')
        ], string="GST Treatment", compute="_compute_l10n_in_gst_treatment", store=True, readonly=False)
    l10n_in_state_id = fields.Many2one('res.country.state', string="Location of supply")
    l10n_in_company_country_code = fields.Char(related='company_id.country_id.code', string="Country code")
    l10n_in_gstin = fields.Char(string="GSTIN")
    # For Export invoice this data is need in GSTR report
    l10n_in_shipping_bill_number = fields.Char('Shipping bill number', readonly=True, states={'draft': [('readonly', False)]})
    l10n_in_shipping_bill_date = fields.Date('Shipping bill date', readonly=True, states={'draft': [('readonly', False)]})
    l10n_in_shipping_port_code_id = fields.Many2one('l10n_in.port.code', 'Port code', states={'draft': [('readonly', False)]})
    l10n_in_reseller_partner_id = fields.Many2one('res.partner', 'Reseller', domain=[('vat', '!=', False)], help="Only Registered Reseller", readonly=True, states={'draft': [('readonly', False)]})

    @api.depends('amount_total')
    def _compute_amount_total_words(self):
        for invoice in self:
            invoice.amount_total_words = invoice.currency_id.amount_to_text(invoice.amount_total)

    @api.depends('partner_id')
    def _compute_l10n_in_gst_treatment(self):
        for record in self:
            record.l10n_in_gst_treatment = record.partner_id.l10n_in_gst_treatment

    @api.model
    def _l10n_in_get_indian_state(self, partner):
        """In tax return filing, If customer is not Indian in that case place of supply is must set to Other Territory.
        So we set Other Territory in l10n_in_state_id when customer(partner) is not Indian
        Also we raise if state is not set in Indian customer.
        State is big role under GST because tax type is depend on.for more information check this https://www.cbic.gov.in/resources//htdocs-cbec/gst/Integrated%20goods%20&%20Services.pdf"""
        if partner.country_id and partner.country_id.code == 'IN' and not partner.state_id:
            raise ValidationError(_("State is missing from address in '%s'. First set state after post this invoice again.", partner.name))
        elif partner.country_id and partner.country_id.code != 'IN':
            return self.env.ref('l10n_in.state_in_ot')
        return partner.state_id


    @api.model
    def _get_tax_grouping_key_from_tax_line(self, tax_line):
        # OVERRIDE to group taxes also by product.
        res = super()._get_tax_grouping_key_from_tax_line(tax_line)
        if tax_line.move_id.journal_id.company_id.country_id.code == 'IN':
            res['product_id'] = tax_line.product_id.id
            res['product_uom_id'] = tax_line.product_uom_id.id
        return res

    @api.model
    def _get_tax_grouping_key_from_base_line(self, base_line, tax_vals):
        # OVERRIDE to group taxes also by product.
        res = super()._get_tax_grouping_key_from_base_line(base_line, tax_vals)
        if base_line.move_id.journal_id.company_id.country_id.code == 'IN':
            res['product_id'] = base_line.product_id.id
            res['product_uom_id'] = base_line.product_uom_id.id
        return res

    @api.model
    def _get_tax_key_for_group_add_base(self, line):
        # DEPRECATED: TO BE REMOVED IN MASTER
        tax_key = super(AccountMove, self)._get_tax_key_for_group_add_base(line)

        tax_key += [
            line.product_id.id,
            line.product_uom_id.id,
        ]
        return tax_key

    def _l10n_in_get_shipping_partner(self):
        """Overwrite in sale"""
        self.ensure_one()
        return self.partner_id

    @api.model
    def _l10n_in_get_shipping_partner_gstin(self, shipping_partner):
        """Overwrite in sale"""
        return shipping_partner.vat

    def _post(self, soft=True):
        """Use journal type to define document type because not miss state in any entry including POS entry"""
        posted = super()._post(soft)
        gst_treatment_name_mapping = {k: v for k, v in
                             self._fields['l10n_in_gst_treatment']._description_selection(self.env)}
        for move in posted.filtered(lambda m: m.l10n_in_company_country_code == 'IN'):
            """Check state is set in company/sub-unit"""
            company_unit_partner = move.journal_id.l10n_in_gstin_partner_id or move.journal_id.company_id
            if not company_unit_partner.state_id:
                raise ValidationError(_(
                    "State is missing from your company/unit %(company_name)s (%(company_id)s).\nFirst set state in your company/unit.",
                    company_name=company_unit_partner.name,
                    company_id=company_unit_partner.id
                ))
            elif move.journal_id.type == 'purchase':
                move.l10n_in_state_id = company_unit_partner.state_id

            shipping_partner = move._l10n_in_get_shipping_partner()
            # In case of shipping address does not have GSTN then also check customer(partner_id) GSTN
            # This happens when Bill-to Ship-to transaction where shipping(Ship-to) address is unregistered and customer(Bill-to) is registred.
            move.l10n_in_gstin = move._l10n_in_get_shipping_partner_gstin(shipping_partner) or move.partner_id.vat
            if not move.l10n_in_gstin and move.l10n_in_gst_treatment in ['regular', 'composition', 'special_economic_zone', 'deemed_export']:
                raise ValidationError(_(
                    "Partner %(partner_name)s (%(partner_id)s) GSTIN is required under GST Treatment %(name)s",
                    partner_name=shipping_partner.name,
                    partner_id=shipping_partner.id,
                    name=gst_treatment_name_mapping.get(move.l10n_in_gst_treatment)
                ))
            if move.journal_id.type == 'sale':
                move.l10n_in_state_id = self._l10n_in_get_indian_state(shipping_partner)
                if not move.l10n_in_state_id:
                    move.l10n_in_state_id = self._l10n_in_get_indian_state(move.partner_id)
                #still state is not set then assumed that transaction is local like PoS so set state of company unit
                if not move.l10n_in_state_id:
                    move.l10n_in_state_id = company_unit_partner.state_id
        return posted

```

## File: models\chart_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    def _prepare_all_journals(self, acc_template_ref, company, journals_dict=None):
        res = super(AccountChartTemplate, self)._prepare_all_journals(acc_template_ref, company, journals_dict=journals_dict)
        if self == self.env.ref('l10n_in.indian_chart_template_standard'):
            for journal in res:
                if journal.get('type') in ('sale','purchase'):
                    journal['l10n_in_gstin_partner_id'] = company.partner_id.id
                if journal['code'] == 'INV':
                    journal['name'] = _('Tax Invoices')
        return res

class AccountTaxTemplate(models.Model):
    _inherit = 'account.tax.template'

    l10n_in_reverse_charge = fields.Boolean("Reverse charge", help="Tick this if this tax is reverse charge. Only for Indian accounting")
    
    def _get_tax_vals(self, company, tax_template_to_tax):
        val = super(AccountTaxTemplate, self)._get_tax_vals(company, tax_template_to_tax)
        if self.tax_group_id:
            val['l10n_in_reverse_charge'] = self.l10n_in_reverse_charge
        return val

```

## File: models\port_code.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class L10nInPortCode(models.Model):
    """Port code must be mentioned in export and import of goods under GST."""
    _name = 'l10n_in.port.code'
    _description = "Indian port code"
    _rec_name = 'code'

    code = fields.Char(string="Port Code", required=True)
    name = fields.Char(string="Port", required=True)
    state_id = fields.Many2one('res.country.state', string="State")

    _sql_constraints = [
        ('code_uniq', 'unique (code)', 'The Port Code must be unique!')
    ]

```

## File: models\product_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    l10n_in_hsn_code = fields.Char(string="HSN/SAC Code", help="Harmonized System Nomenclature/Services Accounting Code")
    l10n_in_hsn_description = fields.Char(string="HSN/SAC Description", help="HSN/SAC description is required if HSN/SAC code is not provided.")

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    group_l10n_in_reseller = fields.Boolean(implied_group='l10n_in.group_l10n_in_reseller', string="Manage Reseller(E-Commerce)")

```

## File: models\res_country_state.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class CountryState(models.Model):
    _inherit = 'res.country.state'

    l10n_in_tin = fields.Char('TIN Number', size=2, help="TIN number-first two digits")

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_in_gst_treatment = fields.Selection([
            ('regular', 'Registered Business - Regular'),
            ('composition', 'Registered Business - Composition'),
            ('unregistered', 'Unregistered Business'),
            ('consumer', 'Consumer'),
            ('overseas', 'Overseas'),
            ('special_economic_zone', 'Special Economic Zone'),
            ('deemed_export', 'Deemed Export'),
        ], string="GST Treatment")

    @api.onchange('company_type')
    def onchange_company_type(self):
        res = super().onchange_company_type()
        if self.country_id and self.country_id.code == 'IN':
            self.l10n_in_gst_treatment = (self.company_type == 'company') and 'regular' or 'consumer'
        return res

    @api.onchange('country_id')
    def _onchange_country_id(self):
        res = super()._onchange_country_id()
        if self.country_id and self.country_id.code != 'IN':
            self.l10n_in_gst_treatment = 'overseas'
        elif self.country_id and self.country_id.code == 'IN':
            self.l10n_in_gst_treatment = (self.company_type == 'company') and 'regular' or 'consumer'
        return res

    @api.onchange('vat')
    def onchange_vat(self):
        if self.vat and self.check_vat_in(self.vat):
            state_id = self.env['res.country.state'].search([('l10n_in_tin', '=', self.vat[:2])], limit=1)
            if state_id:
                self.state_id = state_id

    @api.model
    def _commercial_fields(self):
        res = super()._commercial_fields()
        return res + ['l10n_in_gst_treatment']

```

## File: models\uom_uom.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class UoM(models.Model):
    _inherit = "uom.uom"

    # As per GST Rules you need to Specify UQC given by GST.
    l10n_in_code = fields.Char("Indian GST UQC", help="Unique Quantity Code (UQC) under GST")

```

## File: models\__init__.py

```python
# -*- coding:utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account
from . import account_invoice
from . import chart_template
from . import product_template
from . import port_code
from . import res_config_settings
from . import res_country_state
from . import res_partner
from . import uom_uom

```

## File: report\account_invoice_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools


class L10nInAccountInvoiceReport(models.Model):
    _name = "l10n_in.account.invoice.report"
    _description = "Account Invoice Statistics"
    _auto = False
    _order = 'date desc'

    account_move_id = fields.Many2one('account.move', string="Account Move")
    company_id = fields.Many2one('res.company', string="Company")
    date = fields.Date(string="Accounting Date")
    name = fields.Char(string="Invoice Number")
    partner_id = fields.Many2one('res.partner', string="Customer")
    is_reverse_charge = fields.Char("Reverse Charge")
    l10n_in_gst_treatment = fields.Selection([
            ('regular', 'Registered Business - Regular'),
            ('composition', 'Registered Business - Composition'),
            ('unregistered', 'Unregistered Business'),
            ('consumer', 'Consumer'),
            ('overseas', 'Overseas'),
            ('special_economic_zone', 'Special Economic Zone'),
            ('deemed_export', 'Deemed Export'),
        ], string="GST Treatment")
    journal_id = fields.Many2one('account.journal', string="Journal")
    state = fields.Selection([('draft', 'Unposted'), ('posted', 'Posted')], string='Status')
    igst_amount = fields.Float(string="IGST Amount")
    cgst_amount = fields.Float(string="CGST Amount")
    sgst_amount = fields.Float(string="SGST Amount")
    cess_amount = fields.Float(string="Cess Amount")
    price_total = fields.Float(string='Total Without Tax')
    total = fields.Float(string="Invoice Total")
    reversed_entry_id = fields.Many2one('account.move', string="Refund Invoice", help="From where this Refund is created")
    shipping_bill_number = fields.Char(string="Shipping Bill Number")
    shipping_bill_date = fields.Date(string="Shipping Bill Date")
    shipping_port_code_id = fields.Many2one('l10n_in.port.code', string='Shipping port code')
    ecommerce_partner_id = fields.Many2one('res.partner', string="E-commerce")
    move_type = fields.Selection(selection=[
        ('entry', 'Journal Entry'),
        ('out_invoice', 'Customer Invoice'),
        ('out_refund', 'Customer Credit Note'),
        ('in_invoice', 'Vendor Bill'),
        ('in_refund', 'Vendor Credit Note'),
        ('out_receipt', 'Sales Receipt'),
        ('in_receipt', 'Purchase Receipt')])
    partner_vat = fields.Char(string="Customer GSTIN")
    ecommerce_vat = fields.Char(string="E-commerce GSTIN")
    tax_rate = fields.Float(string="Rate")
    place_of_supply = fields.Char(string="Place of Supply")
    is_pre_gst = fields.Char(string="Is Pre GST")
    is_ecommerce = fields.Char(string="Is E-commerce")
    b2cl_is_ecommerce = fields.Char(string="B2CL Is E-commerce")
    b2cs_is_ecommerce = fields.Char(string="B2CS Is E-commerce")
    supply_type = fields.Char(string="Supply Type")
    export_type = fields.Char(string="Export Type")  # String from GSTR column.
    refund_export_type = fields.Char(string="UR Type")  # String from GSTR column.
    b2b_type = fields.Char(string="B2B Invoice Type")
    refund_invoice_type = fields.Char(string="Document Type")
    gst_format_date = fields.Char(string="Formated Date")
    gst_format_refund_date = fields.Char(string="Formated Refund Date")
    gst_format_shipping_bill_date = fields.Char(string="Formated Shipping Bill Date")
    tax_id = fields.Many2one('account.tax', string="Tax")

    def _select(self):
        select_str = """
            SELECT min(sub.id) as id,
            sub.move_id,
            sub.account_move_id,
            sub.name,
            sub.state,
            sub.partner_id,
            sub.date,
            sub.l10n_in_gst_treatment,
            sub.ecommerce_partner_id,
            sub.shipping_bill_number,
            sub.shipping_bill_date,
            sub.shipping_port_code_id,
            sub.total * sub.b2cs_refund_sign as total,
            sub.journal_id,
            sub.company_id,
            sub.move_type,
            sub.reversed_entry_id,
            sub.partner_vat,
            sub.ecommerce_vat,
            sub.tax_rate as tax_rate,
            (CASE WHEN count(sub.is_reverse_charge) > 0
                THEN 'Y'
                ELSE 'N'
                END) AS is_reverse_charge,
            sub.place_of_supply,
            sub.is_pre_gst,
            sub.is_ecommerce,
            sub.b2cl_is_ecommerce,
            sub.b2cs_is_ecommerce,
            sub.supply_type,
            sub.export_type,
            sub.refund_export_type,
            sub.b2b_type,
            sub.refund_invoice_type,
            sub.gst_format_date,
            sub.gst_format_refund_date,
            sub.gst_format_shipping_bill_date,
            sum(sub.igst_amount) * sub.amount_sign * sub.b2cs_refund_sign AS igst_amount,
            sum(sub.cgst_amount) * sub.amount_sign * sub.b2cs_refund_sign AS cgst_amount,
            sum(sub.sgst_amount) * sub.amount_sign * sub.b2cs_refund_sign AS sgst_amount,
            avg(sub.cess_amount) * sub.amount_sign * sub.b2cs_refund_sign AS cess_amount,
            sum(sub.price_total) * sub.amount_sign * sub.b2cs_refund_sign AS price_total,
            sub.tax_id
        """
        return select_str

    def _sub_select(self):
        sub_select_str = """
            SELECT aml.id AS id,
                aml.move_id,
                aml.partner_id,
                am.id AS account_move_id,
                am.name,
                am.state,
                am.date,
                am.l10n_in_gst_treatment AS l10n_in_gst_treatment,
                am.l10n_in_reseller_partner_id AS ecommerce_partner_id,
                am.l10n_in_shipping_bill_number AS shipping_bill_number,
                am.l10n_in_shipping_bill_date AS shipping_bill_date,
                am.l10n_in_shipping_port_code_id AS shipping_port_code_id,
                ABS(am.amount_total_signed) AS total,
                am.journal_id,
                aj.company_id,
                am.move_type AS move_type,
                am.reversed_entry_id AS reversed_entry_id,
                am.l10n_in_gstin AS partner_vat,
                CASE WHEN rp.vat IS NULL THEN '' ELSE rp.vat END AS ecommerce_vat,
                (CASE WHEN at.l10n_in_reverse_charge = True
                    THEN True
                    ELSE NULL
                    END)  AS is_reverse_charge,
                (CASE WHEN ps.l10n_in_tin IS NOT NULL
                    THEN concat(ps.l10n_in_tin,'-',ps.name)
                    WHEN ps.id IS NULL and cps.l10n_in_tin IS NOT NULL
                    THEN concat(cps.l10n_in_tin,'-',cps.name)
                    ELSE ''
                    END) AS place_of_supply,
                (CASE WHEN am.move_type in ('out_refund', 'in_refund') and refund_am.date <= to_date('2017-07-01', 'YYYY-MM-DD')
                    THEN 'Y'
                    ELSE 'N'
                    END) as is_pre_gst,

                (CASE WHEN am.l10n_in_reseller_partner_id IS NOT NULL
                    THEN 'Y'
                    ELSE 'N'
                    END) as is_ecommerce,
                (CASE WHEN am.l10n_in_reseller_partner_id IS NOT NULL
                    THEN 'Y'
                    ELSE 'N'
                    END) as b2cl_is_ecommerce,
                (CASE WHEN am.l10n_in_reseller_partner_id IS NOT NULL
                    THEN 'E'
                    ELSE 'OE'
                    END) as b2cs_is_ecommerce,
                (CASE WHEN am.l10n_in_state_id = cp.state_id or p.id IS NULL
                    THEN 'Intra State'
                    WHEN am.l10n_in_state_id != cp.state_id and p.id IS NOT NULL
                    THEN 'Inter State'
                    END) AS supply_type,
                (CASE WHEN am.l10n_in_gst_treatment in ('deemed_export', 'overseas') and am.amount_tax > 0.00
                    THEN 'EXPWP'
                    WHEN am.l10n_in_gst_treatment in ('deemed_export', 'overseas') and am.amount_tax <= 0.00
                    THEN 'EXPWOP'
                    ELSE ''
                    END) AS export_type,
                (CASE WHEN am.l10n_in_gst_treatment in ('deemed_export', 'overseas') and am.amount_tax > 0.00
                    THEN 'EXPWP'
                    WHEN am.l10n_in_gst_treatment in ('deemed_export', 'overseas') and am.amount_tax <= 0.00
                    THEN 'EXPWOP'
                    ELSE 'B2CL'
                    END) AS refund_export_type,
                (CASE WHEN am.l10n_in_gst_treatment = 'regular'
                    THEN 'Regular'
                    WHEN am.l10n_in_gst_treatment = 'deemed_export'
                    THEN 'Deemed'
                    WHEN am.l10n_in_gst_treatment = 'overseas' and am.amount_tax > 0.00
                    THEN 'Export with IGST'
                    WHEN am.l10n_in_gst_treatment = 'special_economic_zone' and am.amount_tax > 0.00
                    THEN 'SEZ with IGST payment'
                    WHEN am.l10n_in_gst_treatment = 'special_economic_zone' and am.amount_tax <= 0.00
                    THEN 'SEZ without IGST payment'
                    END) AS b2b_type,
                (CASE WHEN am.move_type = 'out_refund'
                    THEN 'C'
                    WHEN am.move_type = 'in_refund'
                    THEN 'D'
                    ELSE ''
                    END) as refund_invoice_type,
                (CASE WHEN am.date IS NOT NULL
                    THEN TO_CHAR(am.date, 'DD-MON-YYYY')
                    ELSE ''
                    END) as gst_format_date,
                (CASE WHEN refund_am.date IS NOT NULL
                    THEN TO_CHAR(refund_am.date, 'DD-MON-YYYY')
                    ELSE ''
                    END) as gst_format_refund_date,
                (CASE WHEN am.l10n_in_shipping_bill_date IS NOT NULL
                    THEN TO_CHAR(am.l10n_in_shipping_bill_date, 'DD-MON-YYYY')
                    ELSE ''
                    END) as gst_format_shipping_bill_date,
                CASE WHEN tag_rep_ln.account_tax_report_line_id IN
                    (SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name in ('tax_report_line_igst', 'tax_report_line_igst_rc'))
                    THEN aml.balance
                    ELSE 0
                    END AS igst_amount,
                CASE WHEN tag_rep_ln.account_tax_report_line_id IN
                    (SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name in ('tax_report_line_cgst', 'tax_report_line_cgst_rc'))
                    THEN aml.balance
                    ELSE 0
                    END AS cgst_amount,
                CASE WHEN tag_rep_ln.account_tax_report_line_id IN
                    (SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name in ('tax_report_line_sgst', 'tax_report_line_sgst_rc'))
                    THEN aml.balance
                    ELSE 0
                    END AS sgst_amount,
                (WITH account_tax_temp_table AS (
                    SELECT account_account_tag_id
                    FROM account_tax_report_line_tags_rel
                    WHERE account_tax_report_line_id IN
                        (SELECT res_id FROM ir_model_data where module='l10n_in' AND name in ('tax_report_line_cess', 'tax_report_line_cess_rc'))
                    )
                    SELECT sum(temp_aml.balance) from account_move_line temp_aml
                    JOIN account_account_tag_account_move_line_rel aat_aml_rel_temp ON aat_aml_rel_temp.account_move_line_id = temp_aml.id
                    JOIN account_account_tag aat_temp ON aat_temp.id = aat_aml_rel_temp.account_account_tag_id
                    JOIN account_tax_temp_table tag_rep_ln_temp ON aat_temp.id = tag_rep_ln_temp.account_account_tag_id
                    where temp_aml.move_id = aml.move_id and temp_aml.product_id = aml.product_id
                    ) AS cess_amount,
                CASE WHEN tag_rep_ln.account_tax_report_line_id IN
                    (SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name in ('tax_report_line_sgst', 'tax_report_line_sgst_rc'))
                    THEN NULL
                    ELSE (CASE WHEN aml.tax_base_amount <> 0 THEN aml.tax_base_amount * (CASE WHEN aml.balance < 0 THEN -1 ELSE 1 END) ELSE NULL END)
                    END AS price_total,
                (CASE WHEN (aj.type = 'sale' AND am.move_type != 'out_refund') or (aj.type = 'purchase' AND at.l10n_in_reverse_charge AND am.move_type != 'in_refund') THEN -1 ELSE 1 END) AS amount_sign,
                (CASE WHEN am.move_type in ('in_refund','out_refund')
                      AND p.vat IS NULL
                      AND am.l10n_in_gst_treatment != 'overseas'
                      AND (ABS(am.amount_total_signed) <= 250000 OR
                          (ps.id = cp.state_id OR p.id IS NULL))
                      THEN -1
                      ELSE 1 END) AS b2cs_refund_sign,
                (CASE WHEN atr.parent_tax IS NOT NULL THEN atr.parent_tax
                    ELSE at.id END) AS tax_id,
                (CASE WHEN atr.parent_tax IS NOT NULL THEN parent_at.amount
                    ELSE at.amount END) AS tax_rate
        """
        return sub_select_str

    def _from(self):
        from_str = """
            FROM account_move_line aml
                JOIN account_move am ON am.id = aml.move_id
                JOIN account_journal aj ON aj.id = am.journal_id
                JOIN res_company c ON c.id = aj.company_id
                LEFT JOIN account_tax at ON at.id = aml.tax_line_id
                JOIN account_account_tag_account_move_line_rel aat_aml_rel ON aat_aml_rel.account_move_line_id = aml.id
                JOIN account_account_tag aat ON aat.id = aat_aml_rel.account_account_tag_id
                JOIN account_tax_report_line_tags_rel tag_rep_ln ON aat.id = tag_rep_ln.account_account_tag_id
                LEFT JOIN res_partner cp ON cp.id = COALESCE(aj.l10n_in_gstin_partner_id, c.partner_id)
                LEFT JOIN res_country_state cps ON cps.id = cp.state_id
                LEFT JOIN account_move refund_am ON refund_am.id = am.reversed_entry_id
                LEFT JOIN res_partner p ON p.id = aml.partner_id
                LEFT JOIN res_country_state ps ON ps.id = am.l10n_in_state_id
                LEFT JOIN res_partner rp ON rp.id = am.l10n_in_reseller_partner_id
                LEFT JOIN account_tax_filiation_rel atr ON atr.child_tax = at.id
                LEFT JOIN account_tax parent_at ON parent_at.id = atr.parent_tax
                """
        return from_str

    def _where(self):
        return """
                WHERE am.state = 'posted'
                    AND tag_rep_ln.account_tax_report_line_id in (SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name in ('tax_report_line_igst', 'tax_report_line_cgst', 'tax_report_line_sgst', 'tax_report_line_zero_rated', 'tax_report_line_igst_rc', 'tax_report_line_cgst_rc', 'tax_report_line_sgst_rc'))
        """

    def _group_by(self):
        group_by_str = """
        GROUP BY sub.move_id,
            sub.account_move_id,
            sub.name,
            sub.state,
            sub.partner_id,
            sub.date,
            sub.l10n_in_gst_treatment,
            sub.ecommerce_partner_id,
            sub.shipping_bill_number,
            sub.shipping_bill_date,
            sub.shipping_port_code_id,
            sub.total,
            sub.journal_id,
            sub.company_id,
            sub.move_type,
            sub.reversed_entry_id,
            sub.partner_vat,
            sub.ecommerce_vat,
            sub.place_of_supply,
            sub.is_pre_gst,
            sub.is_ecommerce,
            sub.b2cl_is_ecommerce,
            sub.b2cs_is_ecommerce,
            sub.supply_type,
            sub.export_type,
            sub.refund_export_type,
            sub.b2b_type,
            sub.refund_invoice_type,
            sub.gst_format_date,
            sub.gst_format_refund_date,
            sub.gst_format_shipping_bill_date,
            sub.amount_sign,
            sub.tax_id,
            sub.tax_rate,
            sub.b2cs_refund_sign
        """
        return group_by_str

    def init(self):
        tools.drop_view_if_exists(self.env.cr, self._table)
        self.env.cr.execute("""CREATE or REPLACE VIEW %s AS (
            %s
            FROM (
                %s %s %s
            ) AS sub %s)""" % (self._table, self._select(), self._sub_select(),
                self._from(), self._where(), self._group_by()))

```

## File: report\account_payment_report.py

```python
# -*- coding:utf-8 -*-
#az Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools


class L10nInPaymentReport(models.AbstractModel):
    _name = "l10n_in.payment.report"
    _description = "Indian accounting payment report"

    account_move_id = fields.Many2one('account.move', string="Account Move")
    payment_id = fields.Many2one('account.payment', string='Payment')
    currency_id = fields.Many2one('res.currency', string="Currency")
    amount = fields.Float(string="Amount")
    payment_amount = fields.Float(string="Payment Amount")
    partner_id = fields.Many2one('res.partner', string="Customer")
    payment_type = fields.Selection([('outbound', 'Send Money'), ('inbound', 'Receive Money')], string='Payment Type')
    journal_id = fields.Many2one('account.journal', string="Journal")
    company_id = fields.Many2one(related="journal_id.company_id", string="Company")
    place_of_supply = fields.Char(string="Place of Supply")
    supply_type = fields.Char(string="Supply Type")

    l10n_in_tax_id = fields.Many2one('account.tax', string="Tax")
    tax_rate = fields.Float(string="Rate")
    igst_amount = fields.Float(compute="_compute_tax_amount", string="IGST amount")
    cgst_amount = fields.Float(compute="_compute_tax_amount", string="CGST amount")
    sgst_amount = fields.Float(compute="_compute_tax_amount", string="SGST amount")
    cess_amount = fields.Float(compute="_compute_tax_amount", string="CESS amount")
    gross_amount = fields.Float(compute="_compute_tax_amount", string="Gross advance")

    def _compute_l10n_in_tax(self, taxes, price_unit, currency=None, quantity=1.0, product=None, partner=None):
        """common method to compute gst tax amount base on tax group"""
        res = {'igst_amount': 0.0, 'sgst_amount': 0.0, 'cgst_amount': 0.0, 'cess_amount': 0.0}
        AccountTaxRepartitionLine = self.env['account.tax.repartition.line']
        tax_report_line_igst = self.env.ref('l10n_in.tax_report_line_igst', False)
        tax_report_line_cgst = self.env.ref('l10n_in.tax_report_line_cgst', False)
        tax_report_line_sgst = self.env.ref('l10n_in.tax_report_line_sgst', False)
        tax_report_line_cess = self.env.ref('l10n_in.tax_report_line_cess', False)
        filter_tax = taxes.filtered(lambda t: t.type_tax_use != 'none')
        tax_compute = filter_tax.compute_all(price_unit, currency=currency, quantity=quantity, product=product, partner=partner)
        for tax_data in tax_compute['taxes']:
            tax_report_lines = AccountTaxRepartitionLine.browse(tax_data['tax_repartition_line_id']).mapped('tag_ids.tax_report_line_ids')
            if tax_report_line_sgst in tax_report_lines:
                res['sgst_amount'] += tax_data['amount']
            if tax_report_line_cgst in tax_report_lines:
                res['cgst_amount'] += tax_data['amount']
            if tax_report_line_igst in tax_report_lines:
                res['igst_amount'] += tax_data['amount']
            if tax_report_line_cess in tax_report_lines:
                res['cess_amount'] += tax_data['amount']
        res.update(tax_compute)
        return res

    #TO BE OVERWRITTEN
    @api.depends('currency_id')
    def _compute_tax_amount(self):
        """Calculate tax amount base on default tax set in company"""

    def _select(self):
        return """SELECT aml.id AS id,
            aml.move_id as account_move_id,
            ap.id AS payment_id,
            ap.payment_type,
            tax.id as l10n_in_tax_id,
            tax.amount AS tax_rate,
            am.partner_id,
            am.amount_total AS payment_amount,
            am.journal_id,
            aml.currency_id,
            (CASE WHEN ps.l10n_in_tin IS NOT NULL
                THEN concat(ps.l10n_in_tin,'-',ps.name)
                WHEN p.id IS NULL and cps.l10n_in_tin IS NOT NULL
                THEN concat(cps.l10n_in_tin,'-',cps.name)
                ELSE ''
                END) AS place_of_supply,
            (CASE WHEN ps.id = cp.state_id or p.id IS NULL
                THEN 'Intra State'
                WHEN ps.id != cp.state_id and p.id IS NOT NULL
                THEN 'Inter State'
                END) AS supply_type"""

    def _from(self):
        return """FROM account_move_line aml
            JOIN account_move am ON am.id = aml.move_id
            JOIN account_payment ap ON ap.id = aml.payment_id
            JOIN account_account AS ac ON ac.id = aml.account_id
            JOIN account_journal AS aj ON aj.id = am.journal_id
            JOIN res_company AS c ON c.id = aj.company_id
            JOIN account_tax AS tax ON tax.id = (
                CASE WHEN ap.payment_type = 'inbound'
                    THEN c.account_sale_tax_id
                    ELSE c.account_purchase_tax_id END)
            JOIN res_partner p ON p.id = aml.partner_id
            LEFT JOIN res_country_state ps ON ps.id = p.state_id
            LEFT JOIN res_partner cp ON cp.id = COALESCE(aj.l10n_in_gstin_partner_id, c.partner_id)
            LEFT JOIN res_country_state cps ON cps.id = cp.state_id
            """

    def _where(self):
        return """WHERE aml.payment_id IS NOT NULL
            AND tax.tax_group_id in (SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name in ('igst_group','gst_group'))
            AND ac.internal_type IN ('receivable', 'payable') AND am.state = 'posted'"""

    def init(self):
        tools.drop_view_if_exists(self.env.cr, self._table)
        self.env.cr.execute("""CREATE or REPLACE VIEW %s AS (
            %s %s %s)""" % (self._table, self._select(), self._from(), self._where()))


class AdvancesPaymentReport(models.Model):
    _name = "l10n_in.advances.payment.report"
    _inherit = 'l10n_in.payment.report'
    _description = "Advances Payment Analysis"
    _auto = False

    date = fields.Date(string="Payment Date")
    reconcile_amount = fields.Float(string="Reconcile amount in Payment month")

    @api.depends('payment_amount', 'reconcile_amount', 'currency_id')
    def _compute_tax_amount(self):
        """Calculate tax amount base on default tax set in company"""
        account_move_line = self.env['account.move.line']
        for record in self:
            base_amount = record.payment_amount - record.reconcile_amount
            taxes_data = self._compute_l10n_in_tax(
                taxes=record.l10n_in_tax_id,
                price_unit=base_amount,
                currency=record.currency_id or None,
                quantity=1,
                partner=record.partner_id or None)
            record.igst_amount = taxes_data['igst_amount']
            record.cgst_amount = taxes_data['cgst_amount']
            record.sgst_amount = taxes_data['sgst_amount']
            record.cess_amount = taxes_data['cess_amount']
            record.gross_amount = taxes_data['total_excluded']

    def _select(self):
        select_str = super(AdvancesPaymentReport, self)._select()
        select_str += """,
            am.date as date,
            (SELECT sum(amount) FROM account_partial_reconcile AS apr
                WHERE (apr.credit_move_id = aml.id OR apr.debit_move_id = aml.id)
                AND (to_char(apr.max_date, 'MM-YYYY') = to_char(aml.date, 'MM-YYYY'))
            ) AS reconcile_amount,
            (am.amount_total - (SELECT (CASE WHEN SUM(amount) IS NULL THEN 0 ELSE SUM(amount) END) FROM account_partial_reconcile AS apr
                WHERE (apr.credit_move_id = aml.id OR apr.debit_move_id = aml.id)
                AND (to_char(apr.max_date, 'MM-YYYY') = to_char(aml.date, 'MM-YYYY'))
            )) AS amount"""
        return select_str


class L10nInAdvancesPaymentAdjustmentReport(models.Model):
    _name = "l10n_in.advances.payment.adjustment.report"
    _inherit = 'l10n_in.payment.report'
    _description = "Advances Payment Adjustment Analysis"
    _auto = False

    date = fields.Date('Reconcile Date')

    @api.depends('amount', 'currency_id', 'partner_id')
    def _compute_tax_amount(self):
        account_move_line = self.env['account.move.line']
        for record in self:
            taxes_data = self._compute_l10n_in_tax(
                taxes=record.l10n_in_tax_id,
                price_unit=record.amount,
                currency=record.currency_id or None,
                quantity=1,
                partner=record.partner_id or None)
            record.igst_amount = taxes_data['igst_amount']
            record.cgst_amount = taxes_data['cgst_amount']
            record.sgst_amount = taxes_data['sgst_amount']
            record.cess_amount = taxes_data['cess_amount']
            record.gross_amount = taxes_data['total_excluded']

    def _select(self):
        select_str = super(L10nInAdvancesPaymentAdjustmentReport, self)._select()
        select_str += """,
            apr.max_date AS date,
            apr.amount AS amount
            """
        return select_str

    def _from(self):
        from_str = super(L10nInAdvancesPaymentAdjustmentReport, self)._from()
        from_str += """
            JOIN account_partial_reconcile apr ON apr.credit_move_id = aml.id OR apr.debit_move_id = aml.id
            """
        return from_str

    def _where(self):
        where_str = super(L10nInAdvancesPaymentAdjustmentReport, self)._where()
        where_str += """
            AND (apr.max_date > aml.date)
        """
        return where_str

```

## File: report\exempted_gst_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools


class L10nInExemptedReport(models.Model):
    _name = "l10n_in.exempted.report"
    _description = "Exempted Gst Supplied Statistics"
    _auto = False

    account_move_id = fields.Many2one('account.move', string="Account Move")
    partner_id = fields.Many2one('res.partner', string="Customer")
    out_supply_type = fields.Char(string="Outward Supply Type")
    in_supply_type = fields.Char(string="Inward Supply Type")
    nil_rated_amount = fields.Float("Nil rated supplies")
    exempted_amount = fields.Float("Exempted")
    non_gst_supplies = fields.Float("Non GST Supplies")
    date = fields.Date("Date")
    company_id = fields.Many2one('res.company', string="Company")
    journal_id = fields.Many2one('account.journal', string="Journal")

    def _select(self):
        select_str = """SELECT aml.id AS id,
            aml.partner_id AS partner_id,
            am.date,
            aml.balance * (CASE WHEN aj.type = 'sale' THEN -1 ELSE 1 END) AS price_total,
            am.journal_id,
            aj.company_id,
            aml.move_id as account_move_id,

            (CASE WHEN p.state_id = cp.state_id
                THEN (CASE WHEN p.vat IS NOT NULL
                    THEN 'Intra-State supplies to registered persons'
                    ELSE 'Intra-State supplies to unregistered persons'
                    END)
                WHEN p.state_id != cp.state_id
                THEN (CASE WHEN p.vat IS NOT NULL
                    THEN 'Inter-State supplies to registered persons'
                    ELSE 'Inter-State supplies to unregistered persons'
                    END)
            END) AS out_supply_type,
            (CASE WHEN p.state_id = cp.state_id
                THEN 'Intra-State supplies'
                WHEN p.state_id != cp.state_id
                THEN 'Inter-State supplies'
            END) AS in_supply_type,

            (CASE WHEN (
                SELECT MAX(account_tax_id) FROM account_move_line_account_tax_rel
                    JOIN account_tax at ON at.id = account_tax_id
                    WHERE account_move_line_id = aml.id AND at.tax_group_id IN
                     ((SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name='nil_rated_group'))
            ) IS NOT NULL
                THEN aml.balance * (CASE WHEN aj.type = 'sale' THEN -1 ELSE 1 END)
                ELSE 0
            END) AS nil_rated_amount,

            (CASE WHEN (
                SELECT MAX(account_tax_id) FROM account_move_line_account_tax_rel
                    JOIN account_tax at ON at.id = account_tax_id
                    WHERE account_move_line_id = aml.id AND at.tax_group_id IN
                     ((SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name='exempt_group'))
            ) IS NOT NULL
                THEN aml.balance * (CASE WHEN aj.type = 'sale' THEN -1 ELSE 1 END)
                ELSE 0
            END) AS exempted_amount,

            (CASE WHEN (
                SELECT MAX(account_tax_id) FROM account_move_line_account_tax_rel
                    WHERE account_move_line_id = aml.id
                ) IS NULL
                THEN aml.balance * (CASE WHEN aj.type = 'sale' THEN -1 ELSE 1 END)
                ELSE 0
            END) AS non_gst_supplies
        """
        return select_str

    def _from(self):
        from_str = """FROM account_move_line aml
            JOIN account_move am ON am.id = aml.move_id
            JOIN account_account aa ON aa.id = aml.account_id
            JOIN account_journal aj ON aj.id = am.journal_id
            JOIN res_company c ON c.id = aj.company_id
            LEFT JOIN res_partner cp ON cp.id = COALESCE(aj.l10n_in_gstin_partner_id, c.partner_id)
            LEFT JOIN res_partner p ON p.id = am.partner_id
            LEFT JOIN res_country pc ON pc.id = p.country_id
            WHERE aa.internal_type = 'other' and aml.tax_line_id IS NULL
        """
        return from_str

    def init(self):
        tools.drop_view_if_exists(self.env.cr, self._table)
        self._cr.execute("""CREATE OR REPLACE VIEW %s AS (%s %s)""" % (
            self._table, self._select(), self._from()))

```

## File: report\hsn_gst_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, tools


class L10nInProductHsnReport(models.Model):
    _name = "l10n_in.product.hsn.report"
    _description = "Product HSN Statistics"
    _auto = False
    _order = 'date desc'

    account_move_id = fields.Many2one('account.move', string="Account Move")
    partner_id = fields.Many2one('res.partner', string="Customer")
    product_id = fields.Many2one("product.product", string="Product")
    uom_id = fields.Many2one('uom.uom', string="UOM")
    quantity = fields.Float(string="Product Qty")
    date = fields.Date(string="Date")
    price_total = fields.Float(string='Taxable Value')
    total = fields.Float(string="Total Value")
    igst_amount = fields.Float(string="Integrated Tax Amount")
    cgst_amount = fields.Float(string="Central Tax Amount")
    sgst_amount = fields.Float(string="State/UT Tax Amount")
    cess_amount = fields.Float(string="Cess Amount")
    company_id = fields.Many2one('res.company', string="Company")
    journal_id = fields.Many2one('account.journal', string="Journal")

    hsn_code = fields.Char(string="HSN")
    hsn_description = fields.Char(string="HSN description")

    l10n_in_uom_code = fields.Char(string="UQC")

    def _select(self):
        select_str = """SELECT aml.id AS id,
            aml.move_id AS account_move_id,
            aml.partner_id AS partner_id,
            aml.product_id,
            aml.product_uom_id AS uom_id,
            am.date,
            am.journal_id,
            aj.company_id,
            CASE WHEN pt.l10n_in_hsn_code IS NULL THEN '' ELSE pt.l10n_in_hsn_code END AS hsn_code,
            CASE WHEN pt.l10n_in_hsn_description IS NULL THEN '' ELSE pt.l10n_in_hsn_description END AS hsn_description,
            CASE WHEN uom.l10n_in_code IS NULL THEN '' ELSE uom.l10n_in_code END AS l10n_in_uom_code,
            CASE WHEN tag_rep_ln.account_tax_report_line_id IN
                (SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name in ('tax_report_line_sgst', 'tax_report_line_sgst_rc')) OR at.l10n_in_reverse_charge = True
                THEN 0
                ELSE aml.quantity
                END * (CASE WHEN am.move_type in ('in_refund','out_refund') THEN -1 ELSE 1 END)
                AS quantity,
            CASE WHEN tag_rep_ln.account_tax_report_line_id IN
                (SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name in ('tax_report_line_igst', 'tax_report_line_igst_rc'))
                THEN aml.balance * (CASE WHEN aj.type = 'sale' and am.move_type != 'out_refund' THEN -1 ELSE 1 END)
                ELSE 0
                END * (CASE WHEN am.move_type in ('in_refund','out_refund') THEN -1 ELSE 1 END)
                AS igst_amount,
            CASE WHEN tag_rep_ln.account_tax_report_line_id IN
                (SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name in ('tax_report_line_cgst', 'tax_report_line_cgst_rc'))
                THEN aml.balance * (CASE WHEN aj.type = 'sale' and am.move_type != 'out_refund' THEN -1 ELSE 1 END)
                ELSE 0
                END * (CASE WHEN am.move_type in ('in_refund','out_refund') THEN -1 ELSE 1 END)
                AS cgst_amount,
            CASE WHEN tag_rep_ln.account_tax_report_line_id IN
                (SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name in ('tax_report_line_sgst', 'tax_report_line_sgst_rc'))
                THEN aml.balance * (CASE WHEN aj.type = 'sale' and am.move_type != 'out_refund' THEN -1 ELSE 1 END)
                ELSE 0
                END * (CASE WHEN am.move_type in ('in_refund','out_refund') THEN -1 ELSE 1 END)
                AS  sgst_amount,
            CASE WHEN tag_rep_ln.account_tax_report_line_id IN
                (SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name in ('tax_report_line_cess', 'tax_report_line_cess_rc'))
                THEN aml.balance * (CASE WHEN aj.type = 'sale' and am.move_type != 'out_refund' THEN -1 ELSE 1 END)
                ELSE 0
                END * (CASE WHEN am.move_type in ('in_refund','out_refund') THEN -1 ELSE 1 END)
                AS cess_amount,
            CASE WHEN tag_rep_ln.account_tax_report_line_id IN
                (SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name in ('tax_report_line_sgst', 'tax_report_line_sgst_rc'))
                THEN 0
                ELSE (CASE WHEN aml.tax_line_id IS NOT NULL THEN aml.tax_base_amount ELSE aml.balance * (CASE WHEN aj.type = 'sale' THEN -1 ELSE 1 END) END)
                END * (CASE WHEN am.move_type in ('in_refund','out_refund') THEN -1 ELSE 1 END)
                AS price_total,
            ((CASE WHEN tag_rep_ln.account_tax_report_line_id IN
                (SELECT res_id FROM ir_model_data WHERE module='l10n_in' AND name in ('tax_report_line_sgst', 'tax_report_line_sgst_rc'))
                THEN 0
                ELSE (CASE WHEN aml.tax_line_id IS NOT NULL THEN aml.tax_base_amount ELSE 1 END)
                    END) + (aml.balance * (CASE WHEN aj.type = 'sale' and am.move_type != 'out_refund' THEN -1 ELSE 1 END))
                 )* (CASE WHEN am.move_type in ('in_refund','out_refund') THEN -1 ELSE 1 END)
                AS total
        """
        return select_str

    def _from(self):
        from_str = """FROM account_move_line aml
            JOIN account_move am ON am.id = aml.move_id
            JOIN account_account aa ON aa.id = aml.account_id
            JOIN account_journal aj ON aj.id = am.journal_id
            JOIN product_product pp ON pp.id = aml.product_id
            JOIN product_template pt ON pt.id = pp.product_tmpl_id
            LEFT JOIN account_tax at ON at.id = aml.tax_line_id
            LEFT JOIN account_account_tag_account_move_line_rel aat_aml_rel ON aat_aml_rel.account_move_line_id = aml.id
            LEFT JOIN account_account_tag aat ON aat.id = aat_aml_rel.account_account_tag_id
            LEFT JOIN account_tax_report_line_tags_rel tag_rep_ln ON aat.id = tag_rep_ln.account_account_tag_id
            LEFT JOIN account_move_line_account_tax_rel mt ON mt.account_move_line_id = aml.id
            LEFT JOIN uom_uom uom ON uom.id = aml.product_uom_id
            WHERE aa.internal_type = 'other' AND (aml.tax_line_id IS NOT NULL OR mt.account_tax_id IS NULL)
              AND am.state = 'posted'
        """
        return from_str

    def init(self):
        tools.drop_view_if_exists(self.env.cr, self._table)
        self.env.cr.execute("""CREATE OR REPLACE VIEW %s AS (%s %s)""" % (
            self._table, self._select(), self._from()))

```

## File: report\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_invoice_report
from . import account_payment_report
from . import hsn_gst_report
from . import exempted_gst_report

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_port_code_user,port.code.user,model_l10n_in_port_code,base.group_user,1,0,0,0
access_port_code_account_manager,port.code.user,model_l10n_in_port_code,account.group_account_manager,1,1,1,1
access_l10n_in_account_invoice_report,l10n.in.account.invoice.report.user,model_l10n_in_account_invoice_report,,1,0,0,0
access_l10n_in_advances_payment_report,l10n.in.account.move.user.report,model_l10n_in_advances_payment_report,,1,0,0,0
access_l10n_in_advances_payment_adjustment_report,l10n_in.advances.payment.adjustment.report.user,model_l10n_in_advances_payment_adjustment_report,,1,0,0,0
access_l10n_in_product_hsn_report,l10n.in.product.hsn.report.user,model_l10n_in_product_hsn_report,,1,0,0,0
access_l10n_in_exempted_report,l10n_in.exempted.report.user,model_l10n_in_exempted_report,,1,0,0,0

```

## File: security\l10n_in_security.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="res.groups" id="group_l10n_in_reseller">
        <field name="name">Manage Reseller(E-Commerce)</field>
        <field name="category_id" ref="base.module_category_hidden"/>
    </record>
</odoo>

```

## File: views\account_invoice_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="invoice_form_inherit_l10n_in" model="ir.ui.view">
        <field name="name">account.move.form.inherit.l10n.in</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='ref']" position="after">
                <field name="l10n_in_company_country_code" invisible="1"/>
                <field name="l10n_in_gst_treatment"
                    attrs="{'invisible': ['|', ('l10n_in_company_country_code', '!=', 'IN'), ('move_type', '=', 'entry')], 'required': [('l10n_in_company_country_code', '=', 'IN'), ('move_type', '!=', 'entry')], 'readonly': [('state', '!=', 'draft')]}"/>
            </xpath>
            <xpath expr="//page[@id='other_tab']/group[@id='other_tab_group']" position="after">
                <group string="Export India" attrs="{'invisible': ['|', ('l10n_in_gst_treatment', 'not in', ['overseas', 'deemed_export']), ('move_type', 'not in', ['out_invoice', 'out_refund'])]}">
                    <field name="l10n_in_shipping_bill_number"/>
                    <field name="l10n_in_shipping_bill_date"/>
                    <field name="l10n_in_shipping_port_code_id"/>
                </group>
                <group string="Import India" attrs="{'invisible': ['|', ('l10n_in_gst_treatment', 'not in', ['overseas', 'special_economic_zone']), ('move_type', 'not in', ['in_invoice', 'in_refund'])]}">
                    <field name="l10n_in_shipping_bill_number" string="Bill of Entry Number"/>
                    <field name="l10n_in_shipping_bill_date" string="Bill of Entry Date"/>
                    <field name="l10n_in_shipping_port_code_id"/>
                </group>
            </xpath>
            <xpath expr="//field[@name='partner_id']" position="after">
                <field name="l10n_in_reseller_partner_id"
                       groups="l10n_in.group_l10n_in_reseller"
                       attrs="{'invisible': ['|', '|',('move_type', 'not in', ('out_invoice', 'out_refund')), ('l10n_in_company_country_code', '!=', 'IN'), ('move_type', '=', 'entry')]}"
                       />
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\account_journal_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_account_journal_form_inherit_l10n_in" model="ir.ui.view">
        <field name="name">account.journal.form.inherit.l10n.in</field>
        <field name="model">account.journal</field>
        <field name="inherit_id" ref="account.view_account_journal_form"/>
        <field name="arch" type="xml">
            <field name='profit_account_id' position="attributes">
                <attribute name="attrs">{'invisible': ['|', '&amp;', ('country_code', '!=', 'IN'), ('type', '!=', 'cash'), '&amp;', ('country_code', '=', 'IN'), ('type', 'not in', ['bank', 'cash', 'sale', 'purchase'])]}</attribute>
            </field>
            <field name='loss_account_id' position="attributes">
                <attribute name="attrs">{'invisible': ['|', '&amp;', ('country_code', '!=', 'IN'), ('type', '!=', 'cash'), '&amp;', ('country_code', '=', 'IN'), ('type', 'not in', ['bank', 'cash', 'sale', 'purchase'])]}</attribute>
            </field>
            <field name="company_id" position="after">
                <field name="l10n_in_gstin_partner_id" context="{'show_vat':True}" options='{"no_create": True,"always_reload": True}' attrs="{'invisible': [('country_code', '!=', 'IN')]}"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\account_tax_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_tax_form_inherit_l10n_in" model="ir.ui.view">
        <field name="name">account.tax.form.inherit.l10n.in</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
            <field name="include_base_amount" position="after">
                <field name="l10n_in_reverse_charge" attrs="{'invisible':['|', ('amount_type','=', 'group'), ('country_code', '!=', 'IN')]}"/>
            </field>
        </field>
    </record>
</odoo>

```

## File: views\port_code_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="l10n_in_port_code_form_view" model="ir.ui.view">
        <field name="name">l10n_in.port.code.form</field>
        <field name="model">l10n_in.port.code</field>
        <field name="arch" type="xml">
            <form string="India Port Code">
                 <group>
                    <group>
                        <field name="name"/>
                        <field name="code"/>
                    </group>
                    <group>
                        <field name="state_id"/>
                    </group>
                </group>
            </form>
        </field>
    </record>

    <record id="l10n_in_port_code_tree_view" model="ir.ui.view">
        <field name="name">l10n_in.port.code.tree</field>
        <field name="model">l10n_in.port.code</field>
        <field name="arch" type="xml">
            <tree string="India Port Code">
                <field name="name"/>
                <field name="code"/>
                <field name="state_id"/>
            </tree>
        </field>
    </record>

    <record id="l10n_in_port_code_search_view" model="ir.ui.view">
        <field name="name">l10n_in.port.code.search</field>
        <field name="model">l10n_in.port.code</field>
        <field name="arch" type="xml">
            <search string="India Port Code">
                <field name="name" string="Port" filter_domain="['|',('name', 'ilike', self),('code', 'ilike', self)]"/>
                <field name="state_id"/>
                <group expand="0" string="Group By">
                    <filter string="State" name="state" domain="[]" context="{'group_by': 'state_id'}"/>
                </group>
            </search>
        </field>
    </record>

</odoo>

```

## File: views\product_template_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.ui.view" id="l10n_in.product_template_hsn_code">
        <field name="name">l10n_in.product.template.form.hsn_code</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_form_view"/>
        <field name="arch" type="xml">
            <field name="categ_id" position="after">
                <field name="l10n_in_hsn_code"/>
                <field name="l10n_in_hsn_description"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="l10n_in_report_invoice_document_inherit" inherit_id="account.report_invoice_document">

        <xpath expr="//span[@t-field='o.partner_id.vat']" position="attributes">
            <attribute name="t-if">o.company_id.country_id.code != 'IN'</attribute>
        </xpath>
        <xpath expr="//span[@t-field='o.partner_id.vat']" position="after">
            <span t-field="o.l10n_in_gstin" t-if="o.company_id.country_id.code == 'IN'"/>
        </xpath>
        <xpath expr="//t[@t-set='address']" position="inside">
            <t t-if="o.company_id.country_id.code == 'IN' and o.l10n_in_state_id" class="mt16">
                <t t-if="o.move_type in ('in_invoice', 'in_refund')">
                    Destination of supply: <span t-esc="o.l10n_in_state_id.name"/>
                </t>
                <t t-if="o.move_type in ('out_invoice', 'out_refund')">
                    Place of supply: <span t-esc="o.l10n_in_state_id.name"/>
                </t>
            </t>
        </xpath>
        <xpath expr="//p[@t-if='o.narration']" position="before">
            <t t-if="o.company_id.country_id.code == 'IN'">
                <p id="total_in_words" class="mb16">
                    <strong>Total (In Words): </strong>
                    <span t-field="o.amount_total_words"/>
                </p>
            </t>
        </xpath>

        <xpath expr="//table[@name='invoice_line_table']/thead/tr/th[1]" position="after">
            <t t-if="o.company_id.country_id.code == 'IN'">
                <th>HSN/SAC</th>
            </t>
        </xpath>

        <xpath expr="//t[@name='account_invoice_line_accountable']/td[1]" position="after">
            <td t-if="o.company_id.country_id.code == 'IN'">
              <span t-if="line.product_id.l10n_in_hsn_code" t-field="line.product_id.l10n_in_hsn_code"></span>
            </td>
        </xpath>

        <xpath expr="//h2" position="replace">
            <h2>
                <span t-if="o.move_type == 'out_invoice' and o.state == 'posted'" t-field="o.journal_id.name"/>
                <span t-if="o.move_type == 'out_invoice' and o.state == 'draft'">Draft <span t-field="o.journal_id.name"/></span>
                <span t-if="o.move_type == 'out_invoice' and o.state == 'cancel'">Cancelled <span t-field="o.journal_id.name"/></span>
                <span t-if="o.move_type == 'out_refund'">Credit Note</span>
                <span t-if="o.move_type == 'in_refund'">Vendor Credit Note</span>
                <span t-if="o.move_type == 'in_invoice'">Vendor Bill</span>
                <span t-field="o.name"/>
            </h2>
        </xpath>

    </template>

</odoo>

```

## File: views\report_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- get vat from journal_id for all layout -->
    <template id="l10n_in_external_layout" inherit_id="web.external_layout">
        <xpath expr="//t[@t-if='company.external_report_layout_id']" position="before">
            <t t-if="o and 'journal_id' in o and company.country_id.code == 'IN'">
                <t t-set="vat" t-value="o.journal_id.l10n_in_gstin_partner_id.vat"/>
            </t>
            <t t-elif="o and 'l10n_in_journal_id' in o and company.country_id.code == 'IN'">
                <t t-set="vat" t-value="o.l10n_in_journal_id.l10n_in_gstin_partner_id.vat"/>
            </t>
        </xpath>
    </template>

    <template id="l10n_in_external_layout_standard" inherit_id="web.external_layout_standard">
        <xpath expr="//li[@t-if='company.vat']" position="replace">
            <li t-if="vat or company.vat" class="list-inline-item d-inline"><t t-esc="company.country_id.vat_label or 'Tax ID'"/>: <span t-esc="vat or company.vat"/></li>
        </xpath>
    </template>

    <template id="l10n_in_external_layout_clean" inherit_id="web.external_layout_clean">
        <xpath expr="//li[@t-if='company.vat']" position="replace">
            <li t-if="vat or company.vat"><t t-esc="company.country_id.vat_label or 'Tax ID'"/>: <span t-esc="vat or company.vat"/></li>
        </xpath>
    </template>

    <template id="l10n_in_external_layout_boxed" inherit_id="web.external_layout_boxed">
        <xpath expr="//li[@t-if='company.vat']" position="replace">
            <li t-if="vat or company.vat" class="list-inline-item"><t t-esc="company.country_id.vat_label or 'Tax ID'"/>: <span t-esc="vat or company.vat"/></li>
        </xpath>
    </template>

    <template id="l10n_in_external_layout_background" inherit_id="web.external_layout_background">
        <xpath expr="//li[@t-if='company.vat']" position="replace">
            <li t-if="vat or company.vat" class="list-inline-item"><i class="fa fa-building-o" role="img" aria-label="Fiscal number"/><t t-esc="company.country_id.vat_label or 'Tax ID'"/>: <span t-esc="vat or company.vat"/></li>
        </xpath>
    </template>
</odoo>
```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form_inherit_l10n_in" model="ir.ui.view">
        <field name="name">res.config.settings.form.inherit.l10n_in</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <div id="invoicing_settings" position="inside">
                <div class="col-xs-12 col-md-6 o_setting_box"
                    name="ecommerce_reseller_setting"
                    title="Manage Reseller(E-Commerce)"
                    attrs="{'invisible': [('country_code', '!=', 'IN')]}">
                    <div class="o_setting_left_pane">
                        <field name="group_l10n_in_reseller"/>
                    </div>
                    <div class="o_setting_right_pane" name="l10n_eu_service_right_pane">
                        <label for="group_l10n_in_reseller"/>
                        <div class="text-muted">
                            Use this if setup with Reseller(E-Commerce).
                        </div>
                    </div>
                </div>
            </div>
        </field>
    </record>
</odoo>

```

## File: views\res_country_state_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="l10n_in_view_country_state_form_inherit" model="ir.ui.view">
        <field name="name">l10n.in.res.country.state.form.inhert</field>
        <field name="model">res.country.state</field>
        <field name="inherit_id" ref="base.view_country_state_form"/>
        <field name="arch" type="xml">
            <field name="code" position="after">
                <field name="l10n_in_tin" attrs="{'invisible': [('country_id', '!=', %(base.in)d)]}"/>
            </field>
        </field>
    </record>

</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_in_view_partner_form" model="ir.ui.view">
        <field name="name">l10n.in.res.partner.vat.inherit</field>
        <field name="model">res.partner</field>
        <field name="priority" eval="90"/>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="attributes">
                <attribute name="attrs">{'required':[('l10n_in_gst_treatment', 'in', ['regular', 'composition', 'special_economic_zone', 'deemed_export'])], 'readonly': [('parent_id', '!=', False)]}</attribute>
            </xpath>
            <xpath expr="//field[@name='vat']" position="before">
                <field name="l10n_in_gst_treatment" attrs="{'readonly': [('parent_id', '!=', False)]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\uom_uom_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="product_uom_form_view_inherit_l10n_in" model="ir.ui.view">
        <field name="name">uom.uom.form</field>
        <field name="model">uom.uom</field>
        <field name="inherit_id" ref="uom.product_uom_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='category_id']" position="after">
                <field name="l10n_in_code"/>
            </xpath>
        </field>
    </record>
</odoo>

```

