# Odoo Module: l10n_in

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import demo

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
        'data/account_tax_group_data.xml',
        'data/account.account.tag.csv',
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
        'data/account_chart_template_data.xml',
        'report/audit_trail_report_views.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
        'demo/res_partner_demo.xml',
        'demo/product_demo.xml',
        'demo/account_invoice_demo.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.tag.csv

```csv
"id","name","applicability"
"tax_tag_base_sgst","BASE SGST","taxes"
"tax_tag_base_cgst","BASE CGST","taxes"
"tax_tag_base_igst","BASE IGST","taxes"
"tax_tag_non_itc_base_sgst","NON ITC BASE SGST","taxes"
"tax_tag_non_itc_base_cgst","NON ITC BASE CGST","taxes"
"tax_tag_non_itc_base_igst","NON ITC BASE IGST","taxes"
"tax_tag_other_non_itc_base_sgst","Other NON ITC BASE SGST","taxes"
"tax_tag_other_non_itc_base_cgst","Other NON ITC BASE CGST","taxes"
"tax_tag_other_non_itc_base_igst","Other NON ITC BASE IGST","taxes"
"tax_tag_base_cess","BASE CESS","taxes"
"tax_tag_base_state_cess","BASE STATE CESS","taxes"
"tax_tag_non_itc_base_cess","NON ITC BASE CESS","taxes"
"tax_tag_other_non_itc_base_cess","Other NON ITC BASE CESS","taxes"
"tax_tag_exempt","EXEMPT","taxes"
"tax_tag_nil_rated","NIL-RATED","taxes"
"tax_tag_zero_rated","ZERO-RATED","taxes"
"tax_tag_non_gst_supplies","NON GST SUPPLIES","taxes"
"tax_tag_base_sgst_rc","BASE SGST (RC)","taxes"
"tax_tag_base_cgst_rc","BASE CGST (RC)","taxes"
"tax_tag_base_igst_rc","BASE IGST (RC)","taxes"
"tax_tag_base_cess_rc","BASE CESS (RC)","taxes"
"tax_tag_sgst","SGST","taxes"
"tax_tag_cgst","CGST","taxes"
"tax_tag_igst","IGST","taxes"
"tax_tag_non_itc_sgst","NON ITC SGST","taxes"
"tax_tag_non_itc_cgst","NON ITC CGST","taxes"
"tax_tag_non_itc_igst","NON ITC IGST","taxes"
"tax_tag_other_non_itc_sgst","Other NON ITC SGST","taxes"
"tax_tag_other_non_itc_cgst","Other NON ITC CGST","taxes"
"tax_tag_other_non_itc_igst","Other NON ITC IGST","taxes"
"tax_tag_cess","CESS","taxes"
"tax_tag_state_cess","STATE CESS","taxes"
"tax_tag_non_itc_cess","NON ITC CESS","taxes"
"tax_tag_other_non_itc_cess","Other NON ITC CESS","taxes"
"tax_tag_sgst_rc","SGST (RC)","taxes"
"tax_tag_cgst_rc","CGST (RC)","taxes"
"tax_tag_igst_rc","IGST (RC)","taxes"
"tax_tag_non_itc_sgst_rc","NON ITC SGST (RC)","taxes"
"tax_tag_non_itc_cgst_rc","NON ITC CGST (RC)","taxes"
"tax_tag_non_itc_igst_rc","NON ITC IGST (RC)","taxes"
"tax_tag_other_non_itc_sgst_rc","Other NON ITC SGST (RC)","taxes"
"tax_tag_other_non_itc_cgst_rc","Other NON ITC CGST (RC)","taxes"
"tax_tag_other_non_itc_igst_rc","Other NON ITC IGST (RC)","taxes"
"tax_tag_cess_rc","CESS (RC)","taxes"
"tax_tag_non_itc_cess_rc","NON ITC CESS (RC)","taxes"
"tax_tag_other_non_itc_cess_rc","Other NON ITC CESS (RC)","taxes"

```

## File: data\account.account.template.csv

```csv
"id","name","code","account_type","chart_template_id/id","tag_ids/id","reconcile"
"p10031","Inventories","10031","asset_current","l10n_in.indian_chart_template_standard","","False"
"p10040","Debtors","10040","asset_receivable","l10n_in.indian_chart_template_standard","","True"
"p10041","Debtors (PoS)","10041","asset_receivable","l10n_in.indian_chart_template_standard","","True"
"p10051","SGST Receivable","10051","asset_current","l10n_in.indian_chart_template_standard","l10n_in.sgst_tag_account","False"
"p10052","CGST Receivable","10052","asset_current","l10n_in.indian_chart_template_standard","l10n_in.cgst_tag_account","False"
"p10053","IGST Receivable","10053","asset_current","l10n_in.indian_chart_template_standard","l10n_in.igst_tag_account","False"
"p10057","Reverse Charge Tax Receivable","10057","asset_current","l10n_in.indian_chart_template_standard","","False"
"p10054","TDS Receivable","10058","asset_current","l10n_in.indian_chart_template_standard","","False"
"p10059","Tax Current Account - Receivable","10059","asset_current","l10n_in.indian_chart_template_standard","","False"
"p10061","Deposit Account","10061","asset_current","l10n_in.indian_chart_template_standard","","False"
"p10071","Prepaid Insurance","10071","asset_current","l10n_in.indian_chart_template_standard","","False"
"p1011","Buildings","1011","asset_fixed","l10n_in.indian_chart_template_standard","","False"
"p1012","Land","1012","asset_fixed","l10n_in.indian_chart_template_standard","","False"
"p1013","Equipments","1013","asset_fixed","l10n_in.indian_chart_template_standard","","False"
"p1014","Vehicle","1014","asset_fixed","l10n_in.indian_chart_template_standard","","False"
"p1015","Computer/Laptops (Assets)","1015","asset_fixed","l10n_in.indian_chart_template_standard","","False"
"p1016","Furniture","1016","asset_fixed","l10n_in.indian_chart_template_standard","","False"
"p1017","Air Conditionar","1017","asset_fixed","l10n_in.indian_chart_template_standard","","False"
"p1018","Misc Assets","1018","asset_fixed","l10n_in.indian_chart_template_standard","","False"
"p1111","Capital Account","1111","liability_current","l10n_in.indian_chart_template_standard","","False"
"p1112","Reserve And Surplus Account","1112","liability_current","l10n_in.indian_chart_template_standard","","False"
"p11211","Creditors","11211","liability_payable","l10n_in.indian_chart_template_standard","","True"
"p11221","Bank OD Account","11221","liability_current","l10n_in.indian_chart_template_standard","","False"
"p11222","Secured Loan Account","11222","liability_current","l10n_in.indian_chart_template_standard","","False"
"p11223","Unsecured Loan Account","11223","liability_current","l10n_in.indian_chart_template_standard","","False"
"p11231","TDS Payable","11231","liability_current","l10n_in.indian_chart_template_standard","","False"
"p11232","SGST Payable","11232","liability_current","l10n_in.indian_chart_template_standard","l10n_in.sgst_tag_account","False"
"p11233","CGST Payable","11233","liability_current","l10n_in.indian_chart_template_standard","l10n_in.cgst_tag_account","False"
"p11234","IGST Payable","11234","liability_current","l10n_in.indian_chart_template_standard","l10n_in.igst_tag_account","False"
"p11239","Tax Current Account - Payable","11239","liability_current","l10n_in.indian_chart_template_standard","","False"
"p11241","Wages Payable","11241","liability_current","l10n_in.indian_chart_template_standard","","False"
"p11242","Interest Payable","11242","liability_current","l10n_in.indian_chart_template_standard","","False"
"p11243","Notes Payable","11243","liability_current","l10n_in.indian_chart_template_standard","","False"
"p20011","Local Sales","20011","income","l10n_in.indian_chart_template_standard","","False"
"p20012","Retail Sales","20012","income","l10n_in.indian_chart_template_standard","","False"
"p20013","Export Sales","20013","income","l10n_in.indian_chart_template_standard","","False"
"p20021","Local Services","20021","income","l10n_in.indian_chart_template_standard","","False"
"p20022","Export Services","20022","income","l10n_in.indian_chart_template_standard","","False"
"p2010","Interest Revenues","2010","income","l10n_in.indian_chart_template_standard","","False"
"p2011","Gain on Sale of Assets","2011","income","l10n_in.indian_chart_template_standard","","False"
"2012","Write off Income","2012","income","l10n_in.indian_chart_template_standard","","False"
"p2013","Foreign Exchange Profit","2013","income_other","l10n_in.indian_chart_template_standard","","False"
"p2100","Electricity Expense","2100","expense","l10n_in.indian_chart_template_standard","","False"
"p2101","Salary Expense","2101","expense","l10n_in.indian_chart_template_standard","","False"
"p2102","Office Rent","2102","expense","l10n_in.indian_chart_template_standard","","False"
"p2103","House Keeping Expense","2103","expense","l10n_in.indian_chart_template_standard","","False"
"p2104","Postage And Courier Expense","2104","expense","l10n_in.indian_chart_template_standard","","False"
"p2105","Internet Expense","2105","expense","l10n_in.indian_chart_template_standard","","False"
"p2106","Telephone Expense","2106","expense","l10n_in.indian_chart_template_standard","","False"
"p2107","Purchase Expense","2107","expense","l10n_in.indian_chart_template_standard","","False"
"p2108","Computer/Laptop Accessories","2108","expense","l10n_in.indian_chart_template_standard","","False"
"p2109","News Paper And Magazine","2109","expense","l10n_in.indian_chart_template_standard","","False"
"p2110","Business Promotion","2110","expense","l10n_in.indian_chart_template_standard","","False"
"p2111","Entertainment Expense","2111","expense","l10n_in.indian_chart_template_standard","","False"
"p2112","Professional Services","2112","expense","l10n_in.indian_chart_template_standard","","False"
"p2113","Bank Charges","2113","asset_cash","l10n_in.indian_chart_template_standard","","False"
"p2114","Diwali Bonus/Gift","2114","expense","l10n_in.indian_chart_template_standard","","False"
"p2115","Parts Purchase","2115","expense","l10n_in.indian_chart_template_standard","","False"
"p2116","Repairing Expense","2116","expense","l10n_in.indian_chart_template_standard","","False"
"p2117","Foreign Exchange Loss","2117","expense","l10n_in.indian_chart_template_standard","","False"
"p21181","Sales Commission Expense","21181","expense","l10n_in.indian_chart_template_standard","","False"
"p21182","Stationary Expense","21182","expense","l10n_in.indian_chart_template_standard","","False"
"p21183","Travelling Expense","21183","expense","l10n_in.indian_chart_template_standard","","False"
"p2121","Opening Stock","2121","expense","l10n_in.indian_chart_template_standard","","False"
"p2122","Purchase Stock","2122","expense","l10n_in.indian_chart_template_standard","","False"
"p2123","Closing Stock","2123","expense","l10n_in.indian_chart_template_standard","","False"
"p2131","Loss on Sale of Assets","2131","expense","l10n_in.indian_chart_template_standard","","False"
"p2132","Write Off Expense","2132","expense","l10n_in.indian_chart_template_standard","","False"

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

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="sgst_group" model="account.tax.group">
            <field name="name">SGST</field>
            <field name="country_id" ref="base.in"/>
        </record>
        <record id="cgst_group" model="account.tax.group">
            <field name="name">CGST</field>
            <field name="country_id" ref="base.in"/>
        </record>
        <record id="igst_group" model="account.tax.group">
            <field name="name">IGST</field>
            <field name="country_id" ref="base.in"/>
        </record>
        <record id="cess_group" model="account.tax.group">
            <field name="name">CESS</field>
            <field name="country_id" ref="base.in"/>
        </record>
        <record id="gst_group" model="account.tax.group">
            <field name="name">GST</field>
            <field name="country_id" ref="base.in"/>
        </record>
        <record id="exempt_group" model="account.tax.group">
            <field name="name">Exempt</field>
            <field name="country_id" ref="base.in"/>
        </record>
        <record id="nil_rated_group" model="account.tax.group">
            <field name="name">Nil Rated</field>
            <field name="country_id" ref="base.in"/>
        </record>
        <record id="non_gst_supplies_group" model="account.tax.group">
            <field name="name">Non GST Supplies</field>
            <field name="country_id" ref="base.in"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="cess_sale_5" model="account.tax.template">
        <field name="name">CESS Sale 5%</field>
        <field name="description">CESS 5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'tag_ids': [ref('tax_tag_cess')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'tag_ids': [ref('tax_tag_cess')],
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
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'tag_ids': [ref('tax_tag_cess')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'tag_ids': [ref('tax_tag_cess')],
            }),
        ]"/>
    </record>
    <record id="cess_5_plus_1591_sale" model="account.tax.template">
        <field name="name">CESS 5%+1.591</field>
        <field name="description">CESS 5%+1.591</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">group</field>
        <field name="amount">0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('cess_sale_5'),ref('cess_sale_1591'),])]"/>
        <field name="tax_group_id" ref="cess_group"/>
    </record>
    <record id="cess_21_4170_higer_sale" model="account.tax.template">
        <field name="name">CESS 21% or 4.170</field>
        <field name="description">CESS 21% or 4.170</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">code</field>
        <field name="amount">0</field>
        <field name="python_compute">result=base_amount * 0.21
tax=quantity * 4.17
if tax &gt; result:result=tax</field>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'tag_ids': [ref('tax_tag_cess')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'tag_ids': [ref('tax_tag_cess')],
            }),
        ]"/>
    </record>
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_exempt')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_exempt')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_nil_rated')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_nil_rated')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="non_gst_supplies_sale" model="account.tax.template">
        <field name="name">Non GST Supplies</field>
        <field name="description">Non GST Supplies</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="non_gst_supplies_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_non_gst_supplies')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_non_gst_supplies')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_zero_rated')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_zero_rated')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="igst_sale_1" model="account.tax.template">
        <field name="name">IGST 1%</field>
        <field name="description">IGST 1%</field>
        <field name="type_tax_use">sale</field>
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
    </record>
    <record id="igst_sale_2" model="account.tax.template">
        <field name="name">IGST 2%</field>
        <field name="description">IGST 2%</field>
        <field name="type_tax_use">sale</field>
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="amount_type">percent</field>
        <field name="amount">2</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
    </record>
    <record id="igst_sale_28" model="account.tax.template">
        <field name="name">IGST 28%</field>
        <field name="description">IGST 28%</field>
        <field name="type_tax_use">sale</field>
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="amount_type">percent</field>
        <field name="amount">28</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
    </record>
    <record id="igst_sale_18" model="account.tax.template">
        <field name="name">IGST 18%</field>
        <field name="description">IGST 18%</field>
        <field name="type_tax_use">sale</field>
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="amount_type">percent</field>
        <field name="amount">18</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
    </record>
    <record id="igst_sale_12" model="account.tax.template">
        <field name="name">IGST 12%</field>
        <field name="description">IGST 12%</field>
        <field name="type_tax_use">sale</field>
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="amount_type">percent</field>
        <field name="amount">12</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
    </record>
    <record id="igst_sale_5" model="account.tax.template">
        <field name="name">IGST 5%</field>
        <field name="description">IGST 5%</field>
        <field name="type_tax_use">sale</field>
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="igst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
    </record>
    <record id="sgst_sale_0_5" model="account.tax.template">
        <field name="name">SGST Sale 0.5%</field>
        <field name="description">SGST 0.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">0.5</field>
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
    </record>
    <record id="cgst_sale_0_5" model="account.tax.template">
        <field name="name">CGST Sale 0.5%</field>
        <field name="description">CGST 0.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">0.5</field>
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst')],
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
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
    </record>
    <record id="cgst_sale_1_2" model="account.tax.template">
        <field name="name">CGST Sale 1%</field>
        <field name="description">CGST 1%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst')],
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
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
    </record>
    <record id="cgst_sale_14" model="account.tax.template">
        <field name="name">CGST Sale 14%</field>
        <field name="description">CGST 14%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">14</field>
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst')],
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
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
    </record>
    <record id="cgst_sale_9" model="account.tax.template">
        <field name="name">CGST Sale 9%</field>
        <field name="description">CGST 9%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">9</field>
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst')],
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
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
    </record>
    <record id="cgst_sale_6" model="account.tax.template">
        <field name="name">CGST Sale 6%</field>
        <field name="description">CGST 6%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">6</field>
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst')],
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
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="sgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
    </record>
    <record id="cgst_sale_2_5" model="account.tax.template">
        <field name="name">CGST Sale 2.5%</field>
        <field name="description">CGST 2.5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount_type">percent</field>
        <field name="amount">2.5</field>
        <field name="is_base_affected" eval="False"/>
        <field name="include_base_amount" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="cgst_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'tag_ids': [ref('tax_tag_cess')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'tag_ids': [ref('tax_tag_cess')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'tag_ids': [ref('tax_tag_cess')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'tag_ids': [ref('tax_tag_cess')],
            }),
        ]"/>
    </record>
    <record id="cess_5_plus_1591_purchase" model="account.tax.template">
        <field name="name">CESS 5%+1.591</field>
        <field name="description">CESS 5%+1.591</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('cess_purchase_5'),ref('cess_purchase_1591'),])]"/>
        <field name="tax_group_id" ref="cess_group"/>
    </record>
    <record id="cess_21_4170_higer_purchase" model="account.tax.template">
        <field name="name">CESS 21% or 4.170</field>
        <field name="description">CESS 21% or 4.170</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">code</field>
        <field name="amount">0</field>
        <field name="python_compute">result=base_amount * 0.21
tax=quantity * 4.17
if tax &gt; result:result=tax</field>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'tag_ids': [ref('tax_tag_cess')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10055'),
                'tag_ids': [ref('tax_tag_cess')],
            }),
        ]"/>
    </record>
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_exempt')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_exempt')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="nil_rated_purchase" model="account.tax.template">
        <field name="name">Nil Rated</field>
        <field name="description">Nil Rated</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="tax_group_id" ref="nil_rated_group"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_nil_rated')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_nil_rated')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_zero_rated')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_zero_rated')],
            }),
            (0,0, {'repartition_type': 'tax'}),
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'tag_ids': [ref('tax_tag_igst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'tag_ids': [ref('tax_tag_igst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'tag_ids': [ref('tax_tag_igst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'tag_ids': [ref('tax_tag_igst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'tag_ids': [ref('tax_tag_igst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10053'),
                'tag_ids': [ref('tax_tag_igst')],
            }),
        ]"/>
    </record>
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'tag_ids': [ref('tax_tag_sgst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'tag_ids': [ref('tax_tag_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'tag_ids': [ref('tax_tag_cgst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'tag_ids': [ref('tax_tag_sgst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'tag_ids': [ref('tax_tag_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'tag_ids': [ref('tax_tag_cgst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'tag_ids': [ref('tax_tag_sgst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'tag_ids': [ref('tax_tag_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'tag_ids': [ref('tax_tag_cgst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'tag_ids': [ref('tax_tag_sgst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'tag_ids': [ref('tax_tag_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'tag_ids': [ref('tax_tag_cgst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'tag_ids': [ref('tax_tag_sgst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'tag_ids': [ref('tax_tag_cgst')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'tag_ids': [ref('tax_tag_cgst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'tag_ids': [ref('tax_tag_sgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10051'),
                'tag_ids': [ref('tax_tag_sgst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'tag_ids': [ref('tax_tag_cgst')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10052'),
                'tag_ids': [ref('tax_tag_cgst')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'tag_ids': [ref('tax_tag_cess_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'tag_ids': [ref('tax_tag_cess_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'tag_ids': [ref('tax_tag_cess_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'tag_ids': [ref('tax_tag_cess_rc')],
            }),
        ]"/>
    </record>
    <record id="cess_5_plus_1591_purchase_rc" model="account.tax.template">
        <field name="name">CESS 5%+1.591</field>
        <field name="description">CESS 5%+1.591</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">group</field>
        <field name="amount">0</field>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('cess_purchase_5_rc'),ref('cess_purchase_1591_rc'),])]"/>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
    </record>
    <record id="cess_21_4170_higer_purchase_rc" model="account.tax.template">
        <field name="name">CESS 21% or 4.170</field>
        <field name="description">CESS 21% or 4.170</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount_type">code</field>
        <field name="amount">0</field>
        <field name="python_compute">result=base_amount * 0.21
tax=quantity * 4.17
if tax &gt; result:result=tax</field>
        <field name="tax_group_id" ref="cess_group"/>
        <field name="l10n_in_reverse_charge" eval="True"/>
        <field name="chart_template_id" ref="indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'tag_ids': [ref('tax_tag_cess_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cess_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11235'),
                'tag_ids': [ref('tax_tag_cess_rc')],
            }),
        ]"/>
    </record>
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_igst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11234'),
                'tag_ids': [ref('tax_tag_igst_rc')],
            }),
        ]"/>
    </record>
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst_rc')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst_rc')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst_rc')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst_rc')],
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst_rc')],
            }),          ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_sgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11232'),
                'tag_ids': [ref('tax_tag_sgst_rc')],
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
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst_rc')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('tax_tag_base_cgst_rc')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('p10057'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('p11233'),
                'tag_ids': [ref('tax_tag_cgst_rc')],
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
        <menuitem id="account_reports_in_statements_menu" name="India" parent="account.menu_finance_reports" sequence="5"/>

        <record id="indian_chart_template_standard" model="account.chart.template">
            <field name="name">Indian Chart of Accounts - Standard</field>
            <field name="bank_account_code_prefix">1002</field>
            <field name="cash_account_code_prefix">1001</field>
            <field name="transfer_account_code_prefix">1008</field>
            <field name="code_digits">6</field>
            <field name="currency_id" ref="base.INR"/>
            <field name="country_id" ref="base.in"/>
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
            <field name="account_type">asset_current</field>
            <field name="reconcile" eval="False"/>
            <field name="chart_template_id" ref="indian_chart_template_standard"/>
            <field name="tag_ids" eval="[(6,0,[ref('cess_tag_account'),])]"/>
        </record>
        <record id="p10056" model="account.account.template">
            <field name="name">Tax Receivable</field>
            <field name="code">10056</field>
            <field name="account_type">asset_current</field>
            <field name="reconcile" eval="False"/>
            <field name="chart_template_id" ref="indian_chart_template_standard"/>
        </record>
        <record model="account.account.template" id="p11235">
            <field name="name">CESS Payable</field>
            <field name="code">11235</field>
            <field name="account_type">liability_current</field>
            <field name="reconcile" eval="False"/>
            <field name="chart_template_id" ref="indian_chart_template_standard"/>
            <field name="tag_ids" eval="[(6,0,[ref('cess_tag_account'),])]"/>
        </record>
        <record id="p11236" model="account.account.template">
            <field name="name">Tax Payable</field>
            <field name="code">11236</field>
            <field name="account_type">liability_current</field>
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
        <field name="property_tax_payable_account_id" ref="p11239"/>
        <field name="property_tax_receivable_account_id" ref="p10059"/>
        <field name="income_currency_exchange_account_id" ref="p2013"/>
        <field name="expense_currency_exchange_account_id" ref="p2117"/>
        <field name="default_pos_receivable_account_id" ref="p10041"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="p2132"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="2012"/>
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

    <record id="state_in_oc" model="res.country.state">
        <field name="name">Foreign Country</field>
        <field name="code">IN_OC</field>
        <field name="country_id" ref="base.in"/>
        <field name="l10n_in_tin">96</field>
    </record>

    <record id="state_in_la" model="res.country.state">
        <field name="name">Ladakh</field>
        <field name="code">LA</field>
        <field name="country_id" ref="base.in"/>
        <field name="l10n_in_tin">38</field>
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
        <field name="name">Dadra and Nagar Haveli and Daman and Diu</field>
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
    <record id="uom.product_uom_millimeter" model="uom.uom">
        <field name="l10n_in_code">OTH-OTHERS</field>
    </record>
    <record id="uom.product_uom_km" model="uom.uom">
        <field name="l10n_in_code">KME-KILOMETRE</field>
    </record>
    <record id="uom.product_uom_cm" model="uom.uom">
        <field name="l10n_in_code">CMS-CENTIMETERS</field>
    </record>
    <record id="uom.uom_square_meter" model="uom.uom">
        <field name="l10n_in_code">SQM-SQUARE METERS</field>
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
    <record id="uom.product_uom_yard" model="uom.uom">
        <field name="l10n_in_code">YDS-YARDS</field>
    </record>
    <record id="uom.product_uom_mile" model="uom.uom">
        <field name="l10n_in_code">OTH-OTHERS</field>
    </record>
    <record id="uom.uom_square_foot" model="uom.uom">
        <field name="l10n_in_code">SQF-SQUARE FEET</field>
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
        indian_journals = self.filtered(lambda j: j.company_id.account_fiscal_country_id.code == 'IN' and
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
        aml = self.filtered(lambda l: l.company_id.account_fiscal_country_id.code == 'IN' and l.tax_line_id  and l.product_id)
        for move_line in aml:
            base_lines = move_line.move_id.line_ids.filtered(lambda line: move_line.tax_line_id in line.tax_ids and move_line.product_id == line.product_id)
            move_line.tax_base_amount = abs(sum(base_lines.mapped('balance')))
        remaining_aml = self - aml
        if remaining_aml:
            return super(AccountMoveLine, remaining_aml)._compute_tax_base_amount()


class AccountTax(models.Model):
    _inherit = 'account.tax'

    l10n_in_reverse_charge = fields.Boolean("Reverse charge", help="Tick this if this tax is reverse charge. Only for Indian accounting")

    @api.model
    def _get_generation_dict_from_base_line(self, line_vals, tax_vals, force_caba_exigibility=False):
        # EXTENDS account
        # Group taxes also by product.
        res = super()._get_generation_dict_from_base_line(line_vals, tax_vals, force_caba_exigibility=force_caba_exigibility)
        record = line_vals['record']
        if isinstance(record, models.Model)\
                and record._name == 'account.move.line'\
                and record.company_id.account_fiscal_country_id.code == 'IN':
            res['product_id'] = record.product_id.id
            res['product_uom_id'] = record.product_uom_id.id
        return res

    @api.model
    def _get_generation_dict_from_tax_line(self, line_vals):
        # EXTENDS account
        # Group taxes also by product.
        res = super()._get_generation_dict_from_tax_line(line_vals)
        record = line_vals['record']
        if isinstance(record, models.Model)\
                and record._name == 'account.move.line'\
                and record.company_id.account_fiscal_country_id.code == 'IN':
            res['product_id'] = record.product_id.id
            res['product_uom_id'] = record.product_uom_id.id
        return res

```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import logging

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError, RedirectWarning, UserError

_logger = logging.getLogger(__name__)


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
            ('deemed_export', 'Deemed Export'),
            ('uin_holders', 'UIN Holders'),
        ], string="GST Treatment", compute="_compute_l10n_in_gst_treatment", store=True, readonly=False, copy=True)
    l10n_in_state_id = fields.Many2one('res.country.state', string="Place of supply", compute="_compute_l10n_in_state_id", store=True, readonly=False)
    l10n_in_gstin = fields.Char(string="GSTIN")
    # For Export invoice this data is need in GSTR report
    l10n_in_shipping_bill_number = fields.Char('Shipping bill number', readonly=True, states={'draft': [('readonly', False)]})
    l10n_in_shipping_bill_date = fields.Date('Shipping bill date', readonly=True, states={'draft': [('readonly', False)]})
    l10n_in_shipping_port_code_id = fields.Many2one('l10n_in.port.code', 'Port code', readonly=True, states={'draft': [('readonly', False)]})
    l10n_in_reseller_partner_id = fields.Many2one('res.partner', 'Reseller', domain=[('vat', '!=', False)], help="Only Registered Reseller", readonly=True, states={'draft': [('readonly', False)]})
    l10n_in_journal_type = fields.Selection(string="Journal Type", related='journal_id.type')

    @api.depends('amount_total')
    def _compute_amount_total_words(self):
        for invoice in self:
            invoice.amount_total_words = invoice.currency_id.amount_to_text(invoice.amount_total)

    @api.depends('partner_id')
    def _compute_l10n_in_gst_treatment(self):
        indian_invoice = self.filtered(lambda m: m.country_code == 'IN')
        for record in indian_invoice:
            gst_treatment = record.partner_id.l10n_in_gst_treatment
            if not gst_treatment:
                gst_treatment = 'unregistered'
                if record.partner_id.country_id.code == 'IN' and record.partner_id.vat:
                    gst_treatment = 'regular'
                elif record.partner_id.country_id and record.partner_id.country_id.code != 'IN':
                    gst_treatment = 'overseas'
            record.l10n_in_gst_treatment = gst_treatment
        (self - indian_invoice).l10n_in_gst_treatment = False

    @api.depends('partner_id', 'company_id')
    def _compute_l10n_in_state_id(self):
        for move in self:
            if move.country_code == 'IN' and move.journal_id.type == 'sale':
                country_code = move.partner_id.country_id.code
                if country_code == 'IN':
                    move.l10n_in_state_id = move.partner_id.state_id
                elif country_code:
                    move.l10n_in_state_id = self.env.ref('l10n_in.state_in_oc', raise_if_not_found=False)
                else:
                    move.l10n_in_state_id = move.company_id.state_id
            elif move.country_code == 'IN' and move.journal_id.type == 'purchase':
                move.l10n_in_state_id = move.company_id.state_id
            else:
                move.l10n_in_state_id = False

    def _get_name_invoice_report(self):
        if self.country_code == 'IN':
            # TODO: remove the view mode check in master, only for stable releases
            in_invoice_view = self.env.ref('l10n_in.l10n_in_report_invoice_document_inherit', raise_if_not_found=False)
            if (in_invoice_view and in_invoice_view.sudo().mode == "primary"):
                return 'l10n_in.l10n_in_report_invoice_document_inherit'
        return super()._get_name_invoice_report()

    def _post(self, soft=True):
        """Use journal type to define document type because not miss state in any entry including POS entry"""
        posted = super()._post(soft)
        gst_treatment_name_mapping = {k: v for k, v in
                             self._fields['l10n_in_gst_treatment']._description_selection(self.env)}
        for move in posted.filtered(lambda m: m.country_code == 'IN' and m.is_sale_document()):
            """Check state is set in company/sub-unit"""
            company_unit_partner = move.journal_id.l10n_in_gstin_partner_id or move.journal_id.company_id
            if move.l10n_in_state_id and not move.l10n_in_state_id.l10n_in_tin:
                raise UserError(_("Please set a valid TIN Number on the Place of Supply %s", move.l10n_in_state_id.name))
            if not company_unit_partner.state_id:
                msg = _("Your company %s needs to have a correct address in order to validate this invoice.\n"
                "Set the address of your company (Don't forget the State field)") % (company_unit_partner.name)
                action = {
                    "view_mode": "form",
                    "res_model": "res.company",
                    "type": "ir.actions.act_window",
                    "res_id" : move.company_id.id,
                    "views": [[self.env.ref("base.view_company_form").id, "form"]],
                }
                raise RedirectWarning(msg, action, _('Go to Company configuration'))

            move.l10n_in_gstin = move.partner_id.vat
            if not move.l10n_in_gstin and move.l10n_in_gst_treatment in ['regular', 'composition', 'special_economic_zone', 'deemed_export']:
                raise ValidationError(_(
                    "Partner %(partner_name)s (%(partner_id)s) GSTIN is required under GST Treatment %(name)s",
                    partner_name=move.partner_id.name,
                    partner_id=move.partner_id.id,
                    name=gst_treatment_name_mapping.get(move.l10n_in_gst_treatment)
                ))
        return posted

    def _l10n_in_get_warehouse_address(self):
        """Return address where goods are delivered/received for Invoice/Bill"""
        # TO OVERRIDE
        self.ensure_one()
        return False

    @api.ondelete(at_uninstall=False)
    def _unlink_l10n_in_except_once_post(self):
        # Prevent deleting entries once it's posted for Indian Company only
        if any(m.country_code == 'IN' and m.posted_before for m in self) and not self._context.get('force_delete'):
            raise UserError(_("To keep the audit trail, you can not delete journal entries once they have been posted.\nInstead, you can cancel the journal entry."))

    def _can_be_unlinked(self):
        self.ensure_one()
        return (self.country_code != 'IN' or not self.posted_before) and super()._can_be_unlinked()

    def unlink(self):
        # Add logger here becouse in api ondelete account.move.line is deleted and we can't get total amount
        logger_msg = False
        if any(m.country_code == 'IN' and m.posted_before for m in self):
            if self._context.get('force_delete'):
                moves_details = ", ".join("{entry_number} ({move_id}) amount {amount_total} {currency} and partner {partner_name}".format(
                    entry_number=m.name,
                    move_id=m.id,
                    amount_total=m.amount_total,
                    currency=m.currency_id.name,
                    partner_name=m.partner_id.display_name)
                    for m in self)
                logger_msg = 'Force deleted Journal Entries %s by %s (%s)' % (moves_details, self.env.user.name, self.env.user.id)
        res = super().unlink()
        if logger_msg:
            _logger.info(logger_msg)
        return res

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
        return res

    def _load(self, company):
        res = super(AccountChartTemplate, self)._load(company)
        if self == self.env.ref("l10n_in.indian_chart_template_standard"):
            company.write({
                'account_opening_date': fields.Date.context_today(self).replace(month=4, day=1),
                'fiscalyear_last_month': '3',
            })
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

## File: models\mail_message.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from markupsafe import Markup

from odoo import fields, api, models, _
from odoo.exceptions import UserError


class Message(models.Model):
    _inherit = 'mail.message'

    l10n_in_audit_log_preview = fields.Html(string="Description", compute="_compute_l10n_in_audit_log_preview")
    l10n_in_audit_log_account_move_id = fields.Many2one('account.move', string="Journal Entry", compute="_compute_l10n_in_audit_log_document_name", search="_search_l10n_in_audit_log_document_name")

    @api.depends('body', 'subject', 'tracking_value_ids', 'subtype_id')
    def _compute_l10n_in_audit_log_preview(self):
        for message in self:
            title = message.subject or message.preview
            tracking_value_ids = message.sudo().tracking_value_ids
            if not title and tracking_value_ids:
                title = _("Updated")
            elif not title and message.subtype_id and not message.subtype_id.internal:
                title = message.subtype_id.display_name
            audit_log_preview = Markup("<div>%s</div>") % (title)
            for value in tracking_value_ids:
                audit_log_preview += Markup(
                    "<li>%(old_value)s <i class='o_TrackingValue_separator fa fa-long-arrow-right mx-1 text-600' title='%(title)s' role='img' aria-label='%(title)s'></i>%(new_value)s (%(field)s)</li>"
                ) % {
                    'old_value': value._get_old_display_value()[0] or _("None"),
                    'new_value': value._get_new_display_value()[0] or _("None"),
                    'title': _("Changed"),
                    'field': value.field.field_description,
                }
            message.l10n_in_audit_log_preview = audit_log_preview

    @api.depends('model', 'res_id')
    def _compute_l10n_in_audit_log_document_name(self):
        messages_of_account_move = self.filtered(lambda m: m.model == 'account.move' and m.res_id)
        (self - messages_of_account_move).l10n_in_audit_log_account_move_id = False
        moves = self.env['account.move'].search([('id', 'in', messages_of_account_move.mapped('res_id'))])
        moves_by_id = {m.id: m for m in moves}
        for message in messages_of_account_move:
            message.l10n_in_audit_log_account_move_id = moves_by_id.get(message.res_id, False)

    def _search_l10n_in_audit_log_document_name(self, operator, value):
        is_set = False
        if operator == '!=' and isinstance(value, bool):
            is_set = True
        elif operator not in ['=', 'ilike'] or not isinstance(value, str):
            raise UserError(_('Operation not supported'))
        move_domain = [('company_id.account_fiscal_country_id.code', '=', 'IN')]
        if not is_set:
            move_domain += [('name', operator, value)]
        move_query = self.env['account.move']._search(move_domain)
        return [('model', '=', 'account.move'), ('res_id', 'in', move_query)]

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

TEST_GST_NUMBER = "36AABCT1332L011"

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
            ('uin_holders', 'UIN Holders'),
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

    def check_vat_in(self, vat):
        """
            This TEST_GST_NUMBER is used as test credentials for EDI
            but this is not a valid number as per the regular expression
            so TEST_GST_NUMBER is considered always valid
        """
        if vat == TEST_GST_NUMBER:
            return True
        return super().check_vat_in(vat)

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
from . import mail_message

```

## File: report\audit_trail_report_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_message_tree_audit_log" model="ir.ui.view">
        <field name="name">mail.message.tree.inherit.audit.log</field>
        <field name="model">mail.message</field>
        <field name="priority">99</field>
        <field name="arch" type="xml">
            <tree edit="0" delete="0" create="0" action="action_open_document" type="object">
                <field name="res_id" invisible="1"/>
                <field name="date"/>
                <field name="author_id" widget="many2one_avatar"/>
                <field name="l10n_in_audit_log_account_move_id"/>
                <field name="l10n_in_audit_log_preview"/>
            </tree>
        </field>
    </record>

    <record model="ir.ui.view" id="view_message_tree_audit_log_search">
        <field name="name">mail.message.search</field>
        <field name="model">mail.message</field>
        <field name="priority">99</field>
        <field name="arch" type="xml">
            <search string="Messages Search">
                <field name="l10n_in_audit_log_account_move_id"/>
                <field name="author_id"/>
                <field name="date" string="Date"/>
                <filter string="Update Only" name="update_only" domain="[('tracking_value_ids', '!=', False)]"/>
                <group expand="0" string="Group By">
                    <filter string="date" name="group_by_date" domain="[]" context="{'group_by': 'date'}"/>
                </group>
            </search>
        </field>
    </record>

    <record id="action_l10n_in_audit_trail_report" model="ir.actions.act_window">
        <field name="name">Audit trail</field>
        <field name="res_model">mail.message</field>
        <field name="view_id" ref="view_message_tree_audit_log"/>
        <field name="view_mode">tree</field>
        <field name="domain">[
            ('model', '=', 'account.move'),
            ('message_type', '=', 'notification'),
            ('l10n_in_audit_log_account_move_id', '!=', False),
        ]</field>
        <field name="search_view_id" ref="view_message_tree_audit_log_search"/>
    </record>

    <menuitem id="l10n_in_audit_trail_report_menu" name="Audit trail" action="action_l10n_in_audit_trail_report" parent="l10n_in.account_reports_in_statements_menu" sequence="2"
        groups="account.group_account_readonly"/>
</odoo>

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_port_code_user,port.code.user,model_l10n_in_port_code,base.group_user,1,0,0,0
access_port_code_account_manager,port.code.user,model_l10n_in_port_code,account.group_account_manager,1,1,1,1

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

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="6.07" width="50.4" height="33.8" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.65" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="1350" height="900" transform="translate(4.8 6.07) scale(0.04 0.04)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABUcAAAOKCAYAAAC4eeGqAAAACXBIWXMAASioAAEoqAHH7rSeAAAgAElEQVR4XuzdeXSW5Zk/8CuALAKCoiKoqFRcqQjiBiji0mrr0lYFV4qObW07P8W2081pxRm7cNqO6HSmtZ2pgrSWoKJIKS4oqOigaEWsigruIBRJwk4k5PdHBBLyvnmyvEHg/nzO8ZD3ea47eUJezsn5el33XVT5m6gMAAAAAICUXF1Z1CKrBgAAAABgZyQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAkiQcBQAAAACSJBwFAAAAAJIkHAUAAAAAklRUWVlZmVUEAAAAALCTKdI5CgAAAAAkSTgKAAAAACRJOAoAAAAAJEk4CgAAAAAkSTgKAAAAACRJOAoAAAAAJEk4CgAAAAAkSTgKAAAAACRJOAoAAAAAJEk4CgAAAAAkSTgKAAAA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FEkcBQAAAACKJI4CAAAAAEUSRwEAAACAIomjAAAAAECRxFEAAAAAoEjiKAAAAABQJHEUAAAAACiSOAoAAAAAFEkcBQAAAACKJI4CAAAAAEUSRwEAAACAIomjAAAAAECRxFEAAAAAoEjiKAAAAABQJHEUAAAAACiSOAoAAAAAFEkcBQAAAACKJI4CAAAAAEUSRwEAAACAIomjAAAAAECRxFEAAAAAoEjiKAAAAABQJHEUAAAAACiSOAoAAAAAFEkcBQAAAACKJI4CAAAAAEUSRwEAAACAIvWoubCmagYAAAAAYKuy+kurrRwFAAAAAMokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIonGimzwAABHZJREFUAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJHEUQAAAACgSOIoAAAAAFAkcRQAAAAAKJI4CgAAAAAUSRwFAAAAAIokjgIAAAAARRJHAQAAAIAiiaMAAAAAQJF6JGmoGgIAAAAA2Nr8f/20VHR0WVm4AAAAAElFTkSuQmCC"/>
    </g>
  </g>
</svg>

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
                <field name="country_code" invisible="1"/>
                <field name="l10n_in_journal_type" invisible="1"/>
                <field name="l10n_in_state_id" domain="[('country_id.code', '=', 'IN')]" options="{'no_create': True, 'no_open': True}"
                    attrs="{'invisible': ['|', ('country_code', '!=', 'IN'), ('move_type', '=', 'entry')], 'required': [('country_code', '=', 'IN'), ('move_type', '!=', 'entry'), ('l10n_in_journal_type', 'in', ('sale', 'purchase'))], 'readonly': [('state', '!=', 'draft')]}"/>
                <field name="l10n_in_gst_treatment"
                    attrs="{'invisible': ['|', ('country_code', '!=', 'IN'), ('move_type', '=', 'entry')], 'required': [('country_code', '=', 'IN'), ('move_type', '!=', 'entry')], 'readonly': [('state', '!=', 'draft')]}"/>
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
                       attrs="{'invisible': ['|', '|',('move_type', 'not in', ('out_invoice', 'out_refund')), ('country_code', '!=', 'IN'), ('move_type', '=', 'entry')]}"
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
            <field name="is_base_affected" position="after">
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
    <template id="l10n_in_report_invoice_document_inherit" inherit_id="account.report_invoice_document" primary="True">
        <xpath expr="//div[@name='shipping_address_block']" position="inside">
            <div t-if="o.company_id.account_fiscal_country_id.code == 'IN' and o.partner_shipping_id.vat">
                GSTIN: <span t-field="o.partner_shipping_id.vat"/>
            </div>
        </xpath>
        <xpath expr="//div[@name='address_not_same_as_shipping']//t[@t-set='address']" position="inside">
            <t t-call="l10n_in.place_of_supply"/>
        </xpath>
        <xpath expr="//div[@name='address_same_as_shipping']//t[@t-set='address']" position="inside">
            <t t-call="l10n_in.place_of_supply"/>
        </xpath>
        <xpath expr="//div[@name='no_shipping']//t[@t-set='address']" position="inside">
            <t t-call="l10n_in.place_of_supply"/>
        </xpath>
        <xpath expr="//div[@t-if='not is_html_empty(o.narration)']" position="before">
            <t t-if="o.company_id.account_fiscal_country_id.code == 'IN'">
                <p id="total_in_words" class="mb16">
                    <strong>Total (In Words): </strong>
                    <span t-field="o.amount_total_words"/>
                </p>
            </t>
        </xpath>

        <xpath expr="//table[@name='invoice_line_table']/thead/tr/th[1]" position="after">
            <t t-if="o.company_id.account_fiscal_country_id.code == 'IN'">
                <th>HSN/SAC</th>
            </t>
        </xpath>

        <xpath expr="//t[@name='account_invoice_line_accountable']/td[1]" position="after">
            <td t-if="o.company_id.account_fiscal_country_id.code == 'IN'">
              <span t-if="line.product_id.l10n_in_hsn_code" t-field="line.product_id.l10n_in_hsn_code"></span>
            </td>
        </xpath>

        <xpath expr="//h2" position="replace" >
            <t t-if="o.company_id.account_fiscal_country_id.code == 'IN'">
                <h2>
                        <span t-if="o.move_type == 'out_invoice' and o.state == 'posted'" t-field="o.journal_id.name"/>
                        <span t-if="o.move_type == 'out_invoice' and o.state == 'draft'">Draft <span t-field="o.journal_id.name"/></span>
                        <span t-if="o.move_type == 'out_invoice' and o.state == 'cancel'">Cancelled <span t-field="o.journal_id.name"/></span>
                        <span t-if="o.move_type == 'out_refund'">Credit Note</span>
                        <span t-if="o.move_type == 'in_refund'">Vendor Credit Note</span>
                        <span t-if="o.move_type == 'in_invoice'">Vendor Bill</span>
                        <span t-field="o.name"/>
                </h2>
            </t>
            <t t-else="">$0</t>
        </xpath>

    </template>

    <!-- Workarounds for Studio reports, see odoo/odoo#60660 -->
    <template id="report_invoice" inherit_id="account.report_invoice">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-elif="o._get_name_invoice_report() == 'l10n_in.l10n_in_report_invoice_document_inherit'"
               t-call="l10n_in.l10n_in_report_invoice_document_inherit"
               t-lang="lang"/>
        </xpath>
    </template>

    <template id="report_invoice_with_payments" inherit_id="account.report_invoice_with_payments">
        <xpath expr='//t[@t-call="account.report_invoice_document"]' position="after">
            <t t-elif="o._get_name_invoice_report() == 'l10n_in.l10n_in_report_invoice_document_inherit'"
               t-call="l10n_in.l10n_in_report_invoice_document_inherit"
               t-lang="lang"/>
        </xpath>
    </template>

    <template id="place_of_supply">
        <div t-if="o.l10n_in_state_id">
            Place of supply: <span t-out="o.l10n_in_state_id.name" />
        </div>
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
            <t t-if="o and 'journal_id' in o and company.country_id.code == 'IN' and o.journal_id.l10n_in_gstin_partner_id.vat">
                <t t-set="forced_vat" t-value="o.journal_id.l10n_in_gstin_partner_id.vat"/>
            </t>
            <t t-elif="o and 'l10n_in_journal_id' in o and company.country_id.code == 'IN' and o.l10n_in_journal_id.l10n_in_gstin_partner_id.vat">
                <t t-set="forced_vat" t-value="o.l10n_in_journal_id.l10n_in_gstin_partner_id.vat"/>
            </t>
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

    <record id="product_uom_categ_form_view_inherit_l10n_in" model="ir.ui.view">
        <field name="name">uom.category.form</field>
        <field name="model">uom.category</field>
        <field name="inherit_id" ref="uom.product_uom_categ_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='uom_ids']/tree/field[@name='name']" position="after">
                <field name="l10n_in_code"/>
            </xpath>
        </field>
    </record>
</odoo>

```

