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
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Indian - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/india.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['in'],
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
        'account_tax_python',
        'base_vat',
    ],
    'data': [
        'security/l10n_in_security.xml',
        'security/ir.model.access.csv',
        'data/account_tax_report_tcs_data.xml',
        'data/account_tax_report_tds_data.xml',
        'data/account.account.tag.csv',
        'data/l10n_in_chart_data.xml',
        'data/l10n_in.port.code.csv',
        'data/res_country_state_data.xml',
        'data/uom_data.xml',
        'views/account_invoice_views.xml',
        'views/account_journal_views.xml',
        'views/res_config_settings_views.xml',
        'views/product_template_view.xml',
        'views/port_code_views.xml',
        'views/res_company_views.xml',
        'views/report_invoice.xml',
        'views/res_country_state_view.xml',
        'views/res_partner_views.xml',
        'views/account_tax_views.xml',
        'views/uom_uom_views.xml',
        'report/audit_trail_report_views.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
        'demo/product_demo.xml',
    ],
    'license': 'LGPL-3',
    'assets': {
        'web.assets_backend': [
            'l10n_in/static/src/components/**/*',
        ],
    },
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

## File: data\account_tax_report_tcs_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tcs_report" model="account.report">
        <field name="name">TCS Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.in"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tcs_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tcs_report_line_section_206c_1_alfhc" model="account.report.line">
                <field name="name">Section 206C(1): Alcoholic Liquor for human consumption</field>
                <field name="expression_ids">
                    <record id="tcs_report_line_section_206c_1_alfhc_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">206C(1) Alcoholic Liquor</field>
                    </record>
                </field>
            </record>
            <record id="tcs_report_line_section_206c_1_tl" model="account.report.line">
                <field name="name">Section 206C(1): Tendu leaves</field>
                <field name="expression_ids">
                    <record id="tcs_report_line_section_206c_1_tl_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">206C(1) Tendu leaves</field>
                    </record>
                </field>
            </record>
            <record id="tcs_report_line_section_206c_1_touafl" model="account.report.line">
                <field name="name">Section 206C(1): Timber obtained under a forest lease</field>
                <field name="expression_ids">
                    <record id="tcs_report_line_section_206c_1_touafl_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">206C(1) Timber (forest lease)</field>
                    </record>
                </field>
            </record>
            <record id="tcs_report_line_section_206c_1_tobaotuafl" model="account.report.line">
                <field name="name">Section 206C(1): Timber obtained by any mode other than under a forest lease</field>
                <field name="expression_ids">
                    <record id="tcs_report_line_section_206c_1_tobaotuafl_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">206C(1) Timber (other than under a forest lease)</field>
                    </record>
                </field>
            </record>
            <record id="tcs_report_line_section_206c_1_aofpnbtotl" model="account.report.line">
                <field name="name">Section 206C(1): Any other forest produce not being timber or tendu leaves</field>
                <field name="expression_ids">
                    <record id="tcs_report_line_section_206c_1_aofpnbtotl_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">206C(1) other forest produce</field>
                    </record>
                </field>
            </record>
            <record id="tcs_report_line_section_206c_1_s" model="account.report.line">
                <field name="name">Section 206C(1): Scrap</field>
                <field name="expression_ids">
                    <record id="tcs_report_line_section_206c_1_s_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">206C(1) Scrap</field>
                    </record>
                </field>
            </record>
            <record id="tcs_report_line_section_206c_1_mbcoloio" model="account.report.line">
                <field name="name">Section 206C(1): Minrals, being coal or lignite or iron ore</field>
                <field name="expression_ids">
                    <record id="tcs_report_line_section_206c_1_mbcoloio_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">206C(1) Minrals</field>
                    </record>
                </field>
            </record>
            <record id="tcs_report_line_section_206c_1c_pl" model="account.report.line">
                <field name="name">Section 206C(1C): Parking lot</field>
                <field name="expression_ids">
                    <record id="tcs_report_line_section_206c_1c_pl_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">206C(1C) Parking lot</field>
                    </record>
                </field>
            </record>
            <record id="tcs_report_line_section_206c_1c_tp" model="account.report.line">
                <field name="name">Section 206C(1C): Toll plaza</field>
                <field name="expression_ids">
                    <record id="tcs_report_line_section_206c_1c_tp_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">206C(1C) Toll plaza</field>
                    </record>
                </field>
            </record>
            <record id="tcs_report_line_section_206c_1c_maq" model="account.report.line">
                <field name="name">Section 206C(1C): Mining and quarrying</field>
                <field name="expression_ids">
                    <record id="tcs_report_line_section_206c_1c_maq_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">206C(1C) Mining and quarrying</field>
                    </record>
                </field>
            </record>
            <record id="tcs_report_line_section_206c_1f_mv" model="account.report.line">
                <field name="name">Section 206C(1F): Motor Vehicle</field>
                <field name="expression_ids">
                    <record id="tcs_report_line_section_206c_1f_mv_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">206C(1F)</field>
                    </record>
                </field>
            </record>
            <record id="tcs_report_line_section_206c_1g_som" model="account.report.line">
                <field name="name">Section 206C(1G): Sum of money (above 7 lakhs) for remittance out of India</field>
                <field name="expression_ids">
                    <record id="tcs_report_line_section_206c_1g_som_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">206C(1G) remittance out of India</field>
                    </record>
                </field>
            </record>
            <record id="tcs_report_line_section_206c_1g_soaotpp" model="account.report.line">
                <field name="name">Section 206C(1G): Seller of an overseas tour program package</field>
                <field name="expression_ids">
                    <record id="tcs_report_line_section_206c_1g_soaotpp_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">206C(1G) overseas tour program</field>
                    </record>
                </field>
            </record>
            <record id="tcs_report_line_section_206c_1h_sog" model="account.report.line">
                <field name="name">Section 206C(1H): Sale of Goods</field>
                <field name="expression_ids">
                    <record id="tcs_report_line_section_206c_1h_sog_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">206C(1H)</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\account_tax_report_tds_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tds_report" model="account.report">
        <field name="name">TDS Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.in"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tds_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tds_report_line_section_192" model="account.report.line">
                <field name="name">Section 192: Payment of salary</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_192_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">192</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_192a" model="account.report.line">
                <field name="name">Section 192A: Payment of accumulated balance of provident fund which is taxable in the hands of an employee</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_192a_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">192A</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_193" model="account.report.line">
                <field name="name">Section 193: Interest on securities</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_193_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">193</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194" model="account.report.line">
                <field name="name">Section 194: Income by way of dividend</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194a" model="account.report.line">
                <field name="name">Section 194A: Income by way of interest other than &quot;Interest on securities&quot;</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194a_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194A</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194b" model="account.report.line">
                <field name="name">Section 194B: Income by way of winnings from lotteries, crossword puzzles, card games and other games of any sort</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194b_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194B</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194bb" model="account.report.line">
                <field name="name">Section 194BB: Income by way of winnings from horse races</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194bb_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194BB</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194c" model="account.report.line">
                <field name="name">Section 194C: Payment to contractor/sub-contractor</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194c_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194C</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194d" model="account.report.line">
                <field name="name">Section 194D: Insurance commission</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194d_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194D</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194da" model="account.report.line">
                <field name="name">Section 194DA: Payment in respect of life insurance policy</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194da_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194DA</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194e" model="account.report.line">
                <field name="name">Section 194E: Payment to non-resident sportsmen/sports association</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194e_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194E</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194ee" model="account.report.line">
                <field name="name">Section 194EE: Payment in respect of deposit under National Savings scheme</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194ee_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194EE</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194f" model="account.report.line">
                <field name="name">Section 194F: Payment on account of repurchase of unit by Mutual Fund or Unit Trust of India</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194f_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194F</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194g" model="account.report.line">
                <field name="name">Section 194G: Commission, etc., on sale of lottery tickets</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194g_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194G</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194h" model="account.report.line">
                <field name="name">Section 194H: Commission or brokerage</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194h_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194H</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194i" model="account.report.line">
                <field name="name">Section 194-I: Rent</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194i_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194I</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194ia" model="account.report.line">
                <field name="name">Section 194-IA: Payment on transfer of certain immovable property other than agricultural land</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194ia_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194IA</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194ib" model="account.report.line">
                <field name="name">Section 194-IB: Payment of rent by individual or HUF not liable to tax audit</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194ib_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194IB</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194ic" model="account.report.line">
                <field name="name">Section 194-IC: Payment of monetary consideration under Joint Development Agreements</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194ic_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194IC</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194j" model="account.report.line">
                <field name="name">Section 194J: Fees for professional or technical services</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194j_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194J</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194k" model="account.report.line">
                <field name="name">Section 194K: Income in respect of units payable to resident person</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194k_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194K</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194la" model="account.report.line">
                <field name="name">Section 194LA: Payment of compensation on acquisition of certain immovable property</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194la_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194LA</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194lba" model="account.report.line">
                <field name="name">Section 194LBA(1): Business trust shall deduct tax while distributing, any interest received or receivable by it from a SPV or any income received from renting or leasing or letting out any real estate asset owned directly by it, to its unit holders.</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194lba_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194LBA(1)</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194lb" model="account.report.line">
                <field name="name">Section 194LB: Payment of interest on infrastructure debt fund</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194lb_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194LB</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194lbb" model="account.report.line">
                <field name="name">Section 194LBB: Investment fund paying an income to a unit holder [other than income which is exempt under Section 10(23FBB)]</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194lbb_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194LBB</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194lbc" model="account.report.line">
                <field name="name">Section 194LBC: Income in respect of investment made in a securitisation trust (specified in Explanation of section115TCA)</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194lbc_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194LBC</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194m" model="account.report.line">
                <field name="name">Section 194M: Payment of commission (not being insurance commission), brokerage, contractual fee, professional fee to a resident person by an Individual or a HUF who are not liable to deduct TDS under section 194C, 194H, or 194J.</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194m_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194M</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194n" model="account.report.line">
                <field name="name">Section 194N: Cash withdrawal during the previous year from one or more account maintained by a person with a banking company, co-operative society engaged in business of banking or a post office</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194n_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194N</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194o" model="account.report.line">
                <field name="name">Section 194-O: Payment or credit of amount by the e-commerce operator to e-commerce participant</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194o_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194O</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_194q" model="account.report.line">
                <field name="name">Section 194Q: Purchase of goods</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_194q_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">194Q</field>
                    </record>
                </field>
            </record>
            <record id="tds_report_line_section_195" model="account.report.line">
                <field name="name">Section 195: Payment of any other sum to a Non -resident</field>
                <field name="expression_ids">
                    <record id="tds_report_line_section_195_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">195</field>
                    </record>
                </field>
            </record>
        </field>
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

## File: data\template\account.account-in.csv

```csv
"id","name","code","account_type","tag_ids","reconcile"
"p10031","Inventories","10031","asset_current","","False"
"p10040","Debtors","10040","asset_receivable","","True"
"p10041","Debtors (PoS)","10041","asset_receivable","","True"
"p10051","SGST Receivable","10051","asset_current","l10n_in.sgst_tag_account","False"
"p10052","CGST Receivable","10052","asset_current","l10n_in.cgst_tag_account","False"
"p10053","IGST Receivable","10053","asset_current","l10n_in.igst_tag_account","False"
"p10057","Reverse Charge Tax Receivable","10057","asset_current","","False"
"p10054","TDS Receivable","10058","asset_current","","False"
"p10059","Tax Current Account - Receivable","10059","asset_current","","False"
"p10061","Deposit Account","10061","asset_current","","False"
"p10071","Prepaid Insurance","10071","asset_current","","False"
"p1011","Buildings","1011","asset_fixed","","False"
"p1012","Land","1012","asset_fixed","","False"
"p1013","Equipment","1013","asset_fixed","","False"
"p1014","Vehicle","1014","asset_fixed","","False"
"p1015","Computer/Laptops (Assets)","1015","asset_fixed","","False"
"p1016","Furniture","1016","asset_fixed","","False"
"p1017","Air Conditionar","1017","asset_fixed","","False"
"p1018","Misc Assets","1018","asset_fixed","","False"
"p1111","Capital Account","1111","liability_current","","False"
"p1112","Reserve And Surplus Account","1112","liability_current","","False"
"p11211","Creditors","11211","liability_payable","","True"
"p11221","Bank OD Account","11221","liability_current","","False"
"p11222","Secured Loan Account","11222","liability_current","","False"
"p11223","Unsecured Loan Account","11223","liability_current","","False"
"p11231","TDS Payable","11231","liability_current","","False"
"p11232","SGST Payable","11232","liability_current","l10n_in.sgst_tag_account","False"
"p11233","CGST Payable","11233","liability_current","l10n_in.cgst_tag_account","False"
"p11234","IGST Payable","11234","liability_current","l10n_in.igst_tag_account","False"
"p11239","Tax Current Account - Payable","11239","liability_current","","False"
"p11241","Wages Payable","11241","liability_current","","False"
"p11242","Interest Payable","11242","liability_current","","False"
"p11243","Notes Payable","11243","liability_current","","False"
"p20011","Local Sales","20011","income","","False"
"p20012","Retail Sales","20012","income","","False"
"p20013","Export Sales","20013","income","","False"
"p20021","Local Services","20021","income","","False"
"p20022","Export Services","20022","income","","False"
"p2010","Interest Revenues","2010","income","","False"
"p2011","Gain on Sale of Assets","2011","income","","False"
"2012","Write off Income","2012","income","","False"
"p2013","Foreign Exchange Profit","2013","income_other","","False"
"p2100","Electricity Expense","2100","expense","","False"
"p2101","Salary Expense","2101","expense","","False"
"p2102","Office Rent","2102","expense","","False"
"p2103","House Keeping Expense","2103","expense","","False"
"p2104","Postage And Courier Expense","2104","expense","","False"
"p2105","Internet Expense","2105","expense","","False"
"p2106","Telephone Expense","2106","expense","","False"
"p2107","Purchase Expense","2107","expense","","False"
"p2108","Computer/Laptop Accessories","2108","expense","","False"
"p2109","News Paper And Magazine","2109","expense","","False"
"p2110","Business Promotion","2110","expense","","False"
"p2111","Entertainment Expense","2111","expense","","False"
"p2112","Professional Services","2112","expense","","False"
"p2113","Bank Charges","2113","asset_cash","","False"
"p2114","Diwali Bonus/Gift","2114","expense","","False"
"p2115","Parts Purchase","2115","expense","","False"
"p2116","Repairing Expense","2116","expense","","False"
"p2117","Foreign Exchange Loss","2117","expense","","False"
"p21181","Sales Commission Expense","21181","expense","","False"
"p21182","Stationary Expense","21182","expense","","False"
"p21183","Travelling Expense","21183","expense","","False"
"p2121","Opening Stock","2121","expense","","False"
"p2122","Purchase Stock","2122","expense","","False"
"p2123","Closing Stock","2123","expense","","False"
"p2131","Loss on Sale of Assets","2131","expense","","False"
"p2132","Write Off Expense","2132","expense","","False"
"p11244","TDS Deducted","11244","liability_current","","False"
"p11245","TCS Collected","11245","liability_current","","False"
"p10055","CESS Receivable","10055","asset_current","l10n_in.cess_tag_account","False"
"p10056","Tax Receivable","10056","asset_current","","False"
"p11235","CESS Payable","11235","liability_current","l10n_in.cess_tag_account","False"
"p11236","Tax Payable","11236","liability_current","","False"

```

## File: data\template\account.fiscal.position-in.csv

```csv
"id","name","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"fiscal_position_in_reverse_charge_intra","Reverse charge Intra State","sgst_purchase_1","sgst_purchase_1_rc"
"","","sgst_purchase_2","sgst_purchase_2_rc"
"","","sgst_purchase_5","sgst_purchase_5_rc"
"","","sgst_purchase_12","sgst_purchase_12_rc"
"","","sgst_purchase_18","sgst_purchase_18_rc"
"","","sgst_purchase_28","sgst_purchase_28_rc"
"fiscal_position_in_reverse_charge_inter","Reverse charge Inter State","sgst_purchase_1","igst_purchase_1_rc"
"","","sgst_purchase_2","igst_purchase_2_rc"
"","","sgst_purchase_5","igst_purchase_5_rc"
"","","sgst_purchase_12","igst_purchase_12_rc"
"","","sgst_purchase_18","igst_purchase_18_rc"
"","","sgst_purchase_28","igst_purchase_28_rc"

```

## File: data\template\account.tax-in.csv

```csv
"id","name","description","invoice_label","type_tax_use","amount_type","amount","tax_scope","active","tax_group_id","is_base_affected","include_base_amount","children_tax_ids","python_compute","sequence","l10n_in_reverse_charge","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids"
"tcs_1_us_206c_1_alfhc","1% A","TCS @1% u/s 206C(1): Alcoholic Liquor for human consumption","1% TCS 206C(1)","sale","percent","1.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Alcoholic Liquor"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Alcoholic Liquor"
"tcs_5_us_206c_1_alfhc","5% A","TCS @5% u/s 206C(1): Alcoholic Liquor for human consumption","5% TCS 206C(1)","sale","percent","5.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Alcoholic Liquor"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Alcoholic Liquor"
"tcs_5_us_206c_1_tl","5% T L","TCS @5% u/s 206C(1): Tendu leaves","5% TCS 206C(1)","sale","percent","5.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Tendu leaves"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Tendu leaves"
"tcs_2_5_us_206c_1_touafl","2.5% Tim","TCS @2.5% u/s 206C(1): Timber obtained under a forest lease","2.5% TCS 206C(1)","sale","percent","2.5","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Timber (forest lease)"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Timber (forest lease)"
"tcs_5_us_206c_1_touafl","5% Tim","TCS @5% u/s 206C(1): Timber obtained under a forest lease","5% TCS 206C(1)","sale","percent","5.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Timber (forest lease)"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Timber (forest lease)"
"tcs_2_5_us_206c_1_tobamotuafl","2.5% Tim O","TCS @2.5% u/s 206C(1): Timber obtained by any mode other than under a forest lease","2.5% TCS 206C(1)","sale","percent","2.5","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Timber (other than under a forest lease)"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Timber (other than under a forest lease)"
"tcs_5_us_206c_1_tobamotuafl","5% Tim O","TCS @5% u/s 206C(1): Timber obtained by any mode other than under a forest lease","5% TCS 206C(1)","sale","percent","5.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Timber (other than under a forest lease)"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Timber (other than under a forest lease)"
"tcs_2_5_us_206c_1_aofpnbtotl","2.5% F O","TCS @2.5% u/s 206C(1): Any other forest produce not being timber or tendu leaves","2.5% TCS 206C(1)","sale","percent","2.5","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) other forest produce"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) other forest produce"
"tcs_5_us_206c_1_aofpnbtotl","5% F O","TCS @5% u/s 206C(1): Any other forest produce not being timber or tendu leaves","5% TCS 206C(1)","sale","percent","5.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) other forest produce"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) other forest produce"
"tcs_1_us_206c_1_s","1% Sc","TCS @1% u/s 206C(1): Scrap","1% TCS 206C(1)","sale","percent","1.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Scrap"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Scrap"
"tcs_5_us_206c_1_s","5% Sc","TCS @5% u/s 206C(1): Scrap","5% TCS 206C(1)","sale","percent","5.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Scrap"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Scrap"
"tcs_1_us_206c_1_mbcoloio","1% Min","TCS @1% u/s 206C(1): Minrals, being coal or lignite or iron ore","1% TCS 206C(1)","sale","percent","1.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Minrals"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Minrals"
"tcs_5_us_206c_1_mbcoloio","5% Min","TCS @5% u/s 206C(1): Minrals, being coal or lignite or iron ore","5% TCS 206C(1)","sale","percent","5.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Minrals"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Minrals"
"tcs_2_us_206c_1c_pl","2% P","TCS @2% u/s 206C(1C): Parking lot","2% TCS 206C(1C)","sale","percent","2.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1C) Parking lot"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1C) Parking lot"
"tcs_5_us_206c_1c_pl","5% P","TCS @5% u/s 206C(1C): Parking lot","5% TCS 206C(1C)","sale","percent","5.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1C) Parking lot"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1C) Parking lot"
"tcs_2_us_206c_1c_tp","2% T","TCS @2% u/s 206C(1C): Toll plaza","2% TCS 206C(1C)","sale","percent","2.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1C) Toll plaza"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1C) Toll plaza"
"tcs_5_us_206c_1c_tp","5% T","TCS @5% u/s 206C(1C): Toll plaza","5% TCS 206C(1C)","sale","percent","5.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1C) Toll plaza"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1C) Toll plaza"
"tcs_2_us_206c_1c_maq","2% M Q","TCS @2% u/s 206C(1C): Mining and quarrying","2% TCS 206C(1C)","sale","percent","2.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1C) Mining and quarrying"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1C) Mining and quarrying"
"tcs_5_us_206c_1c_maq","5% M Q","TCS @5% u/s 206C(1C): Mining and quarrying","5% TCS 206C(1C)","sale","percent","5.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1C) Mining and quarrying"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1C) Mining and quarrying"
"tcs_1_us_206c_1f_mv","1% M V","TCS @1% u/s 206C(1F): Motor Vehicle","1% TCS 206C(1F)","sale","percent","1.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1F)"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1F)"
"tcs_5_us_206c_1f_mv","5% M V","TCS @5% u/s 206C(1F): Motor Vehicle","5% TCS 206C(1F)","sale","percent","5.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1F)"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1F)"
"tcs_5_us_206c_1g_som","5% R 206C(1G)","TCS @5% u/s 206C(1G): Sum of money (above 7 lakhs) for remittance out of India","5% TCS 206C(1G)","sale","percent","5.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1G) remittance out of India"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1G) remittance out of India"
"tcs_5_us_206c_1g_soaotpp","5% O T 206C(1G)","TCS @5% u/s 206C(1G): Seller of an overseas tour program package","5% TCS 206C(1G)","sale","percent","5.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1G) overseas tour program"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1G) overseas tour program"
"tcs_0_1_us_206c_1h_sog","0.1% G 206C(1H)","TCS @0.1% u/s 206C(1H): Sale of Goods","0.1% TCS 206C(1H)","sale","percent","0.1","consu","","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1H)"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1H)"
"tcs_1_us_206c_1h_sog","1% G 206C(1H)","TCS @1% u/s 206C(1H): Sale of Goods","1% TCS 206C(1H)","sale","percent","1.0","consu","False","tcs_group","","","","","","","100","base","invoice","",""
"","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1H)"
"","","","","","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1H)"
"cess_sale_5","5% CESS S","CESS 5%","CESS 5%","none","percent","5.0","","","cess_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_cess"
"","","","","","","","","","","","","","","","","","tax","invoice","p11235","l10n_in.tax_tag_cess"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cess"
"","","","","","","","","","","","","","","","","","tax","refund","p11235","l10n_in.tax_tag_cess"
"cess_sale_1591","1.591% CESS S","","1591 PER THOUSAND","none","fixed","1.591","","","cess_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_cess"
"","","","","","","","","","","","","","","","","","tax","invoice","p11235","l10n_in.tax_tag_cess"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cess"
"","","","","","","","","","","","","","","","","","tax","refund","p11235","l10n_in.tax_tag_cess"
"cess_5_plus_1591_sale","5%+1.591 CESS S","CESS 5%+1.591","CESS 5%+1.591","sale","group","0.0","","","cess_group","","","cess_sale_5,cess_sale_1591","","","","","","","",""
"cess_21_4170_higer_sale","21% or 4.170 CESS S","CESS 21% or 4.170","CESS 21% or 4.170","sale","code","0.0","","","cess_group","","","","result=base_amount * 0.21; tax=quantity * 4.17; result = tax if (tax > result) else result;","","","","base","invoice","","l10n_in.tax_tag_base_cess"
"","","","","","","","","","","","","","","","","","tax","invoice","p11235","l10n_in.tax_tag_cess"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cess"
"","","","","","","","","","","","","","","","","","tax","refund","p11235","l10n_in.tax_tag_cess"
"exempt_sale","0% Exempt","Exempt","Exempt","sale","percent","0.0","","","exempt_group","","","","","","","","base","invoice","","l10n_in.tax_tag_exempt"
"","","","","","","","","","","","","","","","","","tax","invoice","",""
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_exempt"
"","","","","","","","","","","","","","","","","","tax","refund","",""
"nil_rated_sale","0%","","Nil Rated","sale","percent","0.0","","","nil_rated_group","","","","","","","","base","invoice","","l10n_in.tax_tag_nil_rated"
"","","","","","","","","","","","","","","","","","tax","invoice","",""
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_nil_rated"
"","","","","","","","","","","","","","","","","","tax","refund","",""
"non_gst_supplies_sale","0% NGST","Non GST Supplies","Non GST Supplies","sale","percent","0.0","","","non_gst_supplies_group","","","","","","","","base","invoice","","l10n_in.tax_tag_non_gst_supplies"
"","","","","","","","","","","","","","","","","","tax","invoice","",""
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_non_gst_supplies"
"","","","","","","","","","","","","","","","","","tax","refund","",""
"igst_sale_0","0% IGST","IGST 0%","IGST 0%","sale","percent","0.0","","","igst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_zero_rated"
"","","","","","","","","","","","","","","","","","tax","invoice","",""
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_zero_rated"
"","","","","","","","","","","","","","","","","","tax","refund","",""
"igst_sale_1","1% IGST S","IGST","IGST 1%","sale","percent","1.0","","","igst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11234","l10n_in.tax_tag_igst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","refund","p11234","l10n_in.tax_tag_igst"
"igst_sale_2","2% IGST","IGST","IGST 2%","sale","percent","2.0","","","igst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11234","l10n_in.tax_tag_igst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","refund","p11234","l10n_in.tax_tag_igst"
"igst_sale_28","28% IGST S","IGST","IGST 28%","sale","percent","28.0","","","igst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11234","l10n_in.tax_tag_igst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","refund","p11234","l10n_in.tax_tag_igst"
"igst_sale_18","18% IGST S","IGST","IGST 18%","sale","percent","18.0","","","igst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11234","l10n_in.tax_tag_igst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","refund","p11234","l10n_in.tax_tag_igst"
"igst_sale_12","12% IGST S","IGST","IGST 12%","sale","percent","12.0","","","igst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11234","l10n_in.tax_tag_igst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","refund","p11234","l10n_in.tax_tag_igst"
"igst_sale_5","5% IGST S","IGST","IGST 5%","sale","percent","5.0","","","igst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11234","l10n_in.tax_tag_igst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","refund","p11234","l10n_in.tax_tag_igst"
"sgst_sale_0_5","0.5% SGST S","SGST","SGST 0.5%","none","percent","0.5","","","sgst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11232","l10n_in.tax_tag_sgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","refund","p11232","l10n_in.tax_tag_sgst"
"cgst_sale_0_5","0.5% CGST S","CGST","CGST 0.5%","none","percent","0.5","","","cgst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11233","l10n_in.tax_tag_cgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","refund","p11233","l10n_in.tax_tag_cgst"
"sgst_sale_1","1% GST S","GST","GST 1%","sale","group","1.0","","","gst_group","","","sgst_sale_0_5,cgst_sale_0_5","","","","","","","",""
"sgst_sale_1_2","1% SGST S","SGST ","SGST 1%","none","percent","1.0","","","sgst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11232","l10n_in.tax_tag_sgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","refund","p11232","l10n_in.tax_tag_sgst"
"cgst_sale_1_2","1% CGST S","","CGST 1%","none","percent","1.0","","","cgst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11233","l10n_in.tax_tag_cgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","refund","p11233","l10n_in.tax_tag_cgst"
"sgst_sale_2","2% GST S","GST","GST 2%","sale","group","2.0","","","gst_group","","","sgst_sale_1_2,cgst_sale_1_2","","","","","","","",""
"sgst_sale_14","14% GST S","SGST","SGST 14%","none","percent","14.0","","","sgst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11232","l10n_in.tax_tag_sgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","refund","p11232","l10n_in.tax_tag_sgst"
"cgst_sale_14","14% CGST S","CGST","CGST 14%","none","percent","14.0","","","cgst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11233","l10n_in.tax_tag_cgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","refund","p11233","l10n_in.tax_tag_cgst"
"sgst_sale_28","28% GST S","GST","GST 28%","sale","group","28.0","","","gst_group","","","sgst_sale_14,cgst_sale_14","","","","","","","",""
"sgst_sale_9","9% SGST S","SGST","SGST 9%","none","percent","9.0","","","sgst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11232","l10n_in.tax_tag_sgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","refund","p11232","l10n_in.tax_tag_sgst"
"cgst_sale_9","9% CGST S","CGST","CGST 9%","none","percent","9.0","","","cgst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11233","l10n_in.tax_tag_cgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","refund","p11233","l10n_in.tax_tag_cgst"
"sgst_sale_18","18% GST S","GST","GST 18%","sale","group","18.0","","","gst_group","","","sgst_sale_9,cgst_sale_9","","","","","","","",""
"sgst_sale_6","6% SGST S","SGST","SGST 6%","none","percent","6.0","","","sgst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11232","l10n_in.tax_tag_sgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","refund","p11232","l10n_in.tax_tag_sgst"
"cgst_sale_6","6% CGST S","CGST","CGST 6%","none","percent","6.0","","","cgst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11233","l10n_in.tax_tag_cgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","refund","p11233","l10n_in.tax_tag_cgst"
"sgst_sale_12","12% GST S","GST","GST 12%","sale","group","12.0","","","gst_group","","","sgst_sale_6,cgst_sale_6","","","","","","","",""
"sgst_sale_2_5","2.5% SGST S","SGST","SGST 2.5%","none","percent","2.5","","","sgst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11232","l10n_in.tax_tag_sgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","refund","p11232","l10n_in.tax_tag_sgst"
"cgst_sale_2_5","2.5% CGST S","CGST","CGST 2.5%","none","percent","2.5","","","cgst_group","False","True","","","","","","base","invoice","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p11233","l10n_in.tax_tag_cgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","refund","p11233","l10n_in.tax_tag_cgst"
"sgst_sale_5","5% GST","GST","GST 5%","sale","group","5.0","","","gst_group","","","sgst_sale_2_5,cgst_sale_2_5","","0","","","","","",""
"cess_purchase_5","5% CESS P","CESS","CESS 5%","none","percent","5.0","","","cess_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_cess"
"","","","","","","","","","","","","","","","","","tax","invoice","p10055","l10n_in.tax_tag_cess"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cess"
"","","","","","","","","","","","","","","","","","tax","refund","p10055","l10n_in.tax_tag_cess"
"cess_purchase_1591","1.591% CESS P","","1591 PER THOUSAND","none","fixed","1.591","","","cess_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_cess"
"","","","","","","","","","","","","","","","","","tax","invoice","p10055","l10n_in.tax_tag_cess"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cess"
"","","","","","","","","","","","","","","","","","tax","refund","p10055","l10n_in.tax_tag_cess"
"cess_5_plus_1591_purchase","5%+1.591 CESS P","CESS 5%+1.591","CESS 5%+1.591","purchase","group","0.0","","","cess_group","","","cess_purchase_5,cess_purchase_1591","","","","","","","",""
"cess_21_4170_higer_purchase","21% or 4.170 CESS P","CESS 21% or 4.170","CESS 21% or 4.170","purchase","code","0.0","","","cess_group","","","","result=base_amount * 0.21; tax=quantity * 4.17; result = tax if (tax > result) else result;","","","","base","invoice","","l10n_in.tax_tag_base_cess"
"","","","","","","","","","","","","","","","","","tax","invoice","p10055","l10n_in.tax_tag_cess"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cess"
"","","","","","","","","","","","","","","","","","tax","refund","p10055","l10n_in.tax_tag_cess"
"exempt_purchase","0% Exempt","Exempt","Exempt","purchase","percent","0.0","","","exempt_group","","","","","","","","base","invoice","","l10n_in.tax_tag_exempt"
"","","","","","","","","","","","","","","","","","tax","invoice","",""
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_exempt"
"","","","","","","","","","","","","","","","","","tax","refund","",""
"nil_rated_purchase","0%","","Nil Rated","purchase","percent","0.0","","","nil_rated_group","","","","","","","","base","invoice","","l10n_in.tax_tag_nil_rated"
"","","","","","","","","","","","","","","","","","tax","invoice","",""
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_nil_rated"
"","","","","","","","","","","","","","","","","","tax","refund","",""
"igst_purchase_0","0% IGST","IGST","IGST 0%","purchase","percent","0.0","","","igst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_zero_rated"
"","","","","","","","","","","","","","","","","","tax","invoice","",""
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_zero_rated"
"","","","","","","","","","","","","","","","","","tax","refund","",""
"igst_purchase_1","1% IGST P","IGST","IGST 1%","purchase","percent","1.0","","","igst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10053","l10n_in.tax_tag_igst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","refund","p10053","l10n_in.tax_tag_igst"
"igst_purchase_2","2% IGST P","IGST","IGST 2%","purchase","percent","2.0","","","igst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10053","l10n_in.tax_tag_igst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","refund","p10053","l10n_in.tax_tag_igst"
"igst_purchase_28","28% IGST P","IGST","IGST 28%","purchase","percent","28.0","","","igst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10053","l10n_in.tax_tag_igst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","refund","p10053","l10n_in.tax_tag_igst"
"igst_purchase_18","18% IGST P","IGST","IGST 18%","purchase","percent","18.0","","","igst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10053","l10n_in.tax_tag_igst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","refund","p10053","l10n_in.tax_tag_igst"
"igst_purchase_12","12% IGST P","IGST","IGST 12%","purchase","percent","12.0","","","igst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10053","l10n_in.tax_tag_igst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","refund","p10053","l10n_in.tax_tag_igst"
"igst_purchase_5","5% IGST P","IGST","IGST 5%","purchase","percent","5.0","","","igst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10053","l10n_in.tax_tag_igst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst"
"","","","","","","","","","","","","","","","","","tax","refund","p10053","l10n_in.tax_tag_igst"
"sgst_purchase_0_5","0.5% SGST P","SGST","SGST 0.5%","none","percent","0.5","","","sgst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10051","l10n_in.tax_tag_sgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","refund","p10051","l10n_in.tax_tag_sgst"
"cgst_purchase_0_5","0.5% CGST P","CGST","CGST 0.5%","none","percent","0.5","","","cgst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10052","l10n_in.tax_tag_cgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","refund","p10052","l10n_in.tax_tag_cgst"
"sgst_purchase_1","1% GST P","GST","GST 1%","purchase","group","1.0","","","gst_group","","","sgst_purchase_0_5,cgst_purchase_0_5","","","","","","","",""
"sgst_purchase_1_2","1% SGST P","SGST","SGST 1%","none","percent","1.0","","","sgst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10051","l10n_in.tax_tag_sgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","refund","p10051","l10n_in.tax_tag_sgst"
"cgst_purchase_1_2","1% CGST P","CGST","CGST 1%","none","percent","1.0","","","cgst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10052","l10n_in.tax_tag_cgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","refund","p10052","l10n_in.tax_tag_cgst"
"sgst_purchase_2","2% GST P","GST","GST 2%","purchase","group","2.0","","","gst_group","","","sgst_purchase_1_2,cgst_purchase_1_2","","","","","","","",""
"sgst_purchase_14","14% SGST P","SGST","SGST 14%","none","percent","14.0","","","sgst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10051","l10n_in.tax_tag_sgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","refund","p10051","l10n_in.tax_tag_sgst"
"cgst_purchase_14","14% CGST P","CGST","CGST 14%","none","percent","14.0","","","cgst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10052","l10n_in.tax_tag_cgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","refund","p10052","l10n_in.tax_tag_cgst"
"sgst_purchase_28","28% GST","GST","GST 28%","purchase","group","28.0","","","gst_group","","","sgst_purchase_14,cgst_purchase_14","","","","","","","",""
"sgst_purchase_9","9% SGST P","SGST","SGST 9%","none","percent","9.0","","","sgst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10051","l10n_in.tax_tag_sgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","refund","p10051","l10n_in.tax_tag_sgst"
"cgst_purchase_9","9% CGST P","CGST","CGST 9%","none","percent","9.0","","","cgst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10052","l10n_in.tax_tag_cgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","refund","p10052","l10n_in.tax_tag_cgst"
"sgst_purchase_18","18% GST P","GST","GST 18%","purchase","group","18.0","","","gst_group","","","sgst_purchase_9,cgst_purchase_9","","","","","","","",""
"sgst_purchase_6","6% SGST P","SGST","SGST 6%","none","percent","6.0","","","sgst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10051","l10n_in.tax_tag_sgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","refund","p10051","l10n_in.tax_tag_sgst"
"cgst_purchase_6","6% CGST P","","CGST 6%","none","percent","6.0","","","cgst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10052","l10n_in.tax_tag_cgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","refund","p10052","l10n_in.tax_tag_cgst"
"sgst_purchase_12","12% GST P","GST","GST 12%","purchase","group","12.0","","","gst_group","","","sgst_purchase_6,cgst_purchase_6","","","","","","","",""
"sgst_purchase_2_5","2.5% SGST P","SGST","SGST 2.5%","none","percent","2.5","","","sgst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10051","l10n_in.tax_tag_sgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst"
"","","","","","","","","","","","","","","","","","tax","refund","p10051","l10n_in.tax_tag_sgst"
"cgst_purchase_2_5","2.5% CGST P","CGST","CGST 2.5%","none","percent","2.5","","","cgst_group","","","","","","","","base","invoice","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","invoice","p10052","l10n_in.tax_tag_cgst"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst"
"","","","","","","","","","","","","","","","","","tax","refund","p10052","l10n_in.tax_tag_cgst"
"sgst_purchase_5","5% GST","GST","GST 5%","purchase","group","5.0","","","gst_group","","","sgst_purchase_2_5,cgst_purchase_2_5","","0","","","","","",""
"cess_purchase_5_rc","5% CESS RC","CESS","CESS 5% RC","none","percent","5.0","","","cess_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_cess_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11235","l10n_in.tax_tag_cess_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cess_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11235","l10n_in.tax_tag_cess_rc"
"cess_purchase_1591_rc","1.591% CESS RC","","1591 PER THOUSAND","none","fixed","1.591","","","cess_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_cess_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11235","l10n_in.tax_tag_cess_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cess_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11235","l10n_in.tax_tag_cess_rc"
"cess_5_plus_1591_purchase_rc","5%+1.591 CESS RC","CESS 5%+1.591","CESS 5%+1.591","purchase","group","0.0","","","cess_group","","","cess_purchase_5_rc,cess_purchase_1591_rc","","","True","","","","",""
"cess_21_4170_higer_purchase_rc","21% or 4.170 CESS RC","CESS 21% or 4.170","CESS 21% or 4.170","purchase","code","0.0","","","cess_group","","","","result=base_amount * 0.21; tax=quantity * 4.17; result = tax if (tax > result) else result;","","True","","base","invoice","","l10n_in.tax_tag_base_cess_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11235","l10n_in.tax_tag_cess_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cess_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11235","l10n_in.tax_tag_cess_rc"
"igst_purchase_1_rc","1% IGST RC","IGST (RC)","IGST 1%","purchase","percent","1.0","","","igst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_igst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11234","l10n_in.tax_tag_igst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11234","l10n_in.tax_tag_igst_rc"
"igst_purchase_2_rc","2% IGST RC","IGST (RC)","IGST 2%","purchase","percent","2.0","","","igst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_igst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11234","l10n_in.tax_tag_igst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11234","l10n_in.tax_tag_igst_rc"
"igst_purchase_28_rc","28% IGST RC","IGST (RC)","IGST 28%","purchase","percent","28.0","","","igst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_igst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11234","l10n_in.tax_tag_igst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11234","l10n_in.tax_tag_igst_rc"
"igst_purchase_18_rc","18% IGST RC","IGST (RC)","IGST 18%","purchase","percent","18.0","","","igst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_igst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11234","l10n_in.tax_tag_igst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11234","l10n_in.tax_tag_igst_rc"
"igst_purchase_12_rc","12% IGST RC","IGST (RC)","IGST 12%","purchase","percent","12.0","","","igst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_igst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11234","l10n_in.tax_tag_igst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11234","l10n_in.tax_tag_igst_rc"
"igst_purchase_5_rc","5% IGST RC","IGST (RC)","IGST 5%","purchase","percent","5.0","","","igst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_igst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11234","l10n_in.tax_tag_igst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_igst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11234","l10n_in.tax_tag_igst_rc"
"sgst_purchase_0_5_rc","0.5% SGST RC","SGST (RC)","SGST 0.5%","none","percent","0.5","","","sgst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_sgst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11232","l10n_in.tax_tag_sgst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11232","l10n_in.tax_tag_sgst_rc"
"cgst_purchase_0_5_rc","0.5% CGST RC","CGST (RC)","CGST 0.5%","none","percent","0.5","","","cgst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_cgst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11233","l10n_in.tax_tag_cgst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11233","l10n_in.tax_tag_cgst_rc"
"sgst_purchase_1_rc","1% GST RC","GST (RC)","GST 1%","purchase","group","1.0","","","gst_group","","","sgst_purchase_0_5_rc,cgst_purchase_0_5_rc","","","True","","","","",""
"sgst_purchase_1_2_rc","1% SGST RC","SGST (RC)","SGST 1%","none","percent","1.0","","","sgst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_sgst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11232","l10n_in.tax_tag_sgst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11232","l10n_in.tax_tag_sgst_rc"
"cgst_purchase_1_2_rc","1% CGST RC","CGST (RC)","CGST 1%","none","percent","1.0","","","cgst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_cgst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11233","l10n_in.tax_tag_cgst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11233","l10n_in.tax_tag_cgst_rc"
"sgst_purchase_2_rc","2% GST RC","GST (RC)","GST 2%","purchase","group","2.0","","","gst_group","","","sgst_purchase_1_2_rc,cgst_purchase_1_2_rc","","","True","","","","",""
"sgst_purchase_14_rc","14% SGST RC","SGST (RC)","SGST 14%","none","percent","14.0","","","sgst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_sgst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11232","l10n_in.tax_tag_sgst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11232","l10n_in.tax_tag_sgst_rc"
"cgst_purchase_14_rc","14% CGST RC","CGST (RC)","CGST 14%","none","percent","14.0","","","cgst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_cgst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11233","l10n_in.tax_tag_cgst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11233","l10n_in.tax_tag_cgst_rc"
"sgst_purchase_28_rc","28% GST RC","GST (RC)","GST 28%","purchase","group","28.0","","","gst_group","","","sgst_purchase_14_rc,cgst_purchase_14_rc","","","True","","","","",""
"sgst_purchase_9_rc","9% SGST RC","SGST (RC)","SGST 9%","none","percent","9.0","","","sgst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_sgst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11232","l10n_in.tax_tag_sgst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11232","l10n_in.tax_tag_sgst_rc"
"cgst_purchase_9_rc","9% CGST RC","CGST (RC)","CGST 9%","none","percent","9.0","","","cgst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_cgst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11233","l10n_in.tax_tag_cgst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11233","l10n_in.tax_tag_cgst_rc"
"sgst_purchase_18_rc","18% GST RC","GST (RC)","GST 18%","purchase","group","18.0","","","gst_group","","","sgst_purchase_9_rc,cgst_purchase_9_rc","","","True","","","","",""
"sgst_purchase_6_rc","6% SGST RC","SGST (RC)","SGST 6%","none","percent","6.0","","","sgst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_sgst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11232","l10n_in.tax_tag_sgst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11232","l10n_in.tax_tag_sgst_rc"
"cgst_purchase_6_rc","6% CGST RC","CGST (RC)","CGST 6%","none","percent","6.0","","","cgst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_cgst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11233","l10n_in.tax_tag_cgst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11233","l10n_in.tax_tag_cgst_rc"
"sgst_purchase_12_rc","12% GST RC","GST (RC)","GST 12%","purchase","group","12.0","","","gst_group","","","sgst_purchase_6_rc,cgst_purchase_6_rc","","","True","","","","",""
"sgst_purchase_2_5_rc","2.5% SGST RC","SGST (RC)","SGST 2.5%","none","percent","2.5","","","sgst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_sgst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11232","l10n_in.tax_tag_sgst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_sgst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11232","l10n_in.tax_tag_sgst_rc"
"cgst_purchase_2_5_rc","2.5% CGST RC","CGST (RC)","CGST 2.5%","none","percent","2.5","","","cgst_group","","","","","","True","","base","invoice","","l10n_in.tax_tag_base_cgst_rc"
"","","","","","","","","","","","","","","","","","tax","invoice","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","invoice","p11233","l10n_in.tax_tag_cgst_rc"
"","","","","","","","","","","","","","","","","","base","refund","","l10n_in.tax_tag_base_cgst_rc"
"","","","","","","","","","","","","","","","","","tax","refund","p10057",""
"","","","","","","","","","","","","","","","","-100","tax","refund","p11233","l10n_in.tax_tag_cgst_rc"
"sgst_purchase_5_rc","5% GST RC","GST (RC)","GST 5%","purchase","group","5.0","","","gst_group","","","sgst_purchase_2_5_rc,cgst_purchase_2_5_rc","","0","True","","","","",""

```

## File: data\template\account.tax.group-in.csv

```csv
"id","name","country_id","sequence","tax_payable_account_id","tax_receivable_account_id"
"sgst_group","SGST","base.in","","p11239","p10059"
"cgst_group","CGST","base.in","","p11239","p10059"
"igst_group","IGST","base.in","","p11239","p10059"
"cess_group","CESS","base.in","","p11239","p10059"
"gst_group","GST","base.in","","p11239","p10059"
"exempt_group","Exempt","base.in","","p11239","p10059"
"nil_rated_group","Nil Rated","base.in","","p11239","p10059"
"non_gst_supplies_group","Non GST Supplies","base.in","","p11239","p10059"
"tcs_group","TCS","base.in","100","p11239","p10059"
"tds_group","TDS","base.in","100","p11239","p10059"

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

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import base64
import logging

from collections import defaultdict

from odoo import api, fields, models, _
from odoo.exceptions import ValidationError, RedirectWarning, UserError
from odoo.tools import frozendict
from odoo.tools.image import image_data_uri

_logger = logging.getLogger(__name__)


class AccountMove(models.Model):
    _inherit = "account.move"

    l10n_in_gst_treatment = fields.Selection([
            ('regular', 'Registered Business - Regular'),
            ('composition', 'Registered Business - Composition'),
            ('unregistered', 'Unregistered Business'),
            ('consumer', 'Consumer'),
            ('overseas', 'Overseas'),
            ('special_economic_zone', 'Special Economic Zone'),
            ('deemed_export', 'Deemed Export'),
            ('uin_holders', 'UIN Holders'),
        ], string="GST Treatment", compute="_compute_l10n_in_gst_treatment", store=True, readonly=False, copy=True, precompute=True)
    l10n_in_state_id = fields.Many2one('res.country.state', string="Place of supply",
        compute="_compute_l10n_in_state_id", store=True, readonly=False, precompute=True)
    l10n_in_gstin = fields.Char(string="GSTIN")
    # For Export invoice this data is need in GSTR report
    l10n_in_shipping_bill_number = fields.Char('Shipping bill number')
    l10n_in_shipping_bill_date = fields.Date('Shipping bill date')
    l10n_in_shipping_port_code_id = fields.Many2one('l10n_in.port.code', 'Port code')
    l10n_in_reseller_partner_id = fields.Many2one('res.partner', 'Reseller', domain=[('vat', '!=', False)], help="Only Registered Reseller")
    l10n_in_journal_type = fields.Selection(string="Journal Type", related='journal_id.type')

    @api.depends('partner_id', 'partner_id.l10n_in_gst_treatment')
    def _compute_l10n_in_gst_treatment(self):
        indian_invoice = self.filtered(lambda m: m.country_code == 'IN')
        for record in indian_invoice:
            if record.state == 'draft':
                gst_treatment = record.partner_id.l10n_in_gst_treatment
                if not gst_treatment:
                    gst_treatment = 'unregistered'
                    if record.partner_id.country_id.code == 'IN' and record.partner_id.vat:
                        gst_treatment = 'regular'
                    elif record.partner_id.country_id and record.partner_id.country_id.code != 'IN':
                        gst_treatment = 'overseas'
                record.l10n_in_gst_treatment = gst_treatment
        (self - indian_invoice).l10n_in_gst_treatment = False

    @api.depends('partner_id', 'partner_shipping_id', 'company_id')
    def _compute_l10n_in_state_id(self):
        for move in self:
            if move.country_code == 'IN' and move.is_sale_document(include_receipts=True):
                partner_state = (
                    move.partner_id.commercial_partner_id == move.partner_shipping_id.commercial_partner_id
                    and move.partner_shipping_id.state_id
                    or move.partner_id.state_id
                )
                if not partner_state:
                    partner_state = move.partner_id.commercial_partner_id.state_id or move.company_id.state_id
                country_code = partner_state.country_id.code or move.country_code
                if country_code == 'IN':
                    move.l10n_in_state_id = partner_state
                else:
                    move.l10n_in_state_id = self.env.ref('l10n_in.state_in_oc', raise_if_not_found=False)
            elif move.country_code == 'IN' and move.journal_id.type == 'purchase':
                move.l10n_in_state_id = move.company_id.state_id
            else:
                move.l10n_in_state_id = False

    @api.depends('l10n_in_state_id', 'l10n_in_gst_treatment')
    def _compute_fiscal_position_id(self):

        def _get_fiscal_state(move, foreign_state):
            """
            Maps each move to its corresponding fiscal state based on its type,
            fiscal conditions, and the state of the associated partner or company.
            """

            if (
                move.country_code != 'IN'
                or not move.is_invoice(include_receipts=True)
                # Partner's FP takes precedence through super
                or move.partner_shipping_id.property_account_position_id
                or move.partner_id.property_account_position_id
            ):
                return False
            elif move.l10n_in_gst_treatment == 'special_economic_zone':
                # Special Economic Zone
                return foreign_state
            elif move.is_sale_document(include_receipts=True):
                # In Sales Documents: Compare place of supply with company state
                return move.l10n_in_state_id if move.l10n_in_state_id.l10n_in_tin != '96' else foreign_state
            elif move.is_purchase_document(include_receipts=True) and move.partner_id.country_id.code == 'IN':
                # In Purchases Documents: Compare place of supply with vendor state
                pos_state_id = move.l10n_in_state_id
                if pos_state_id.l10n_in_tin == '96':
                    return foreign_state
                elif pos_state_id == move.partner_id.state_id:
                    # Intra-State: Group by state matching the company's state.
                    return move.company_id.state_id
                elif pos_state_id != move.partner_id.state_id:
                    # Inter-State: Group by state that doesn't match the company's state.
                    return (
                        pos_state_id == move.company_id.state_id
                        and move.partner_id.state_id
                        or pos_state_id
                    )
            return False

        FiscalPosition = self.env['account.fiscal.position']
        # To avoid ORM call in loops, we are passing the `foreign_state` as parameter
        foreign_state = self.env['res.country.state'].search([('code', '!=', 'IN')], limit=1)
        for state_id, moves in self.grouped(lambda move: _get_fiscal_state(move, foreign_state)).items():
            if state_id:
                virtual_partner = self.env['res.partner'].new({
                    'state_id': state_id.id,
                    'country_id': state_id.country_id.id,
                })
                # Group moves by company to avoid multi-company conflicts
                for company_id, company_moves in moves.grouped('company_id').items():
                    company_moves.fiscal_position_id = FiscalPosition.with_company(
                        company_id
                    )._get_fiscal_position(virtual_partner)
            else:
                super(AccountMove, moves)._compute_fiscal_position_id()

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
            if move.l10n_in_state_id and not move.l10n_in_state_id.l10n_in_tin:
                raise UserError(_("Please set a valid TIN Number on the Place of Supply %s", move.l10n_in_state_id.name))
            if not move.company_id.state_id:
                msg = _("Your company %s needs to have a correct address in order to validate this invoice.\n"
                "Set the address of your company (Don't forget the State field)", move.company_id.name)
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

    def _generate_qr_code(self, silent_errors=False):
        self.ensure_one()
        if self.company_id.country_code == 'IN' and self.company_id.l10n_in_upi_id:
            payment_url = 'upi://pay?pa=%s&pn=%s&am=%s&tr=%s&tn=%s' % (
                self.company_id.l10n_in_upi_id,
                self.company_id.name,
                self.amount_residual,
                self.payment_reference or self.name,
                ("Payment for %s" % self.name))
            barcode = self.env['ir.actions.report'].barcode(barcode_type="QR", value=payment_url, width=120, height=120)
            return image_data_uri(base64.b64encode(barcode))
        return super()._generate_qr_code(silent_errors)

    def _l10n_in_get_hsn_summary_table(self):
        self.ensure_one()
        display_uom = self.env.user.user_has_groups('uom.group_uom')
        tag_igst = self.env.ref('l10n_in.tax_tag_igst')
        tag_cgst = self.env.ref('l10n_in.tax_tag_cgst')
        tag_sgst = self.env.ref('l10n_in.tax_tag_sgst')
        tag_cess = self.env.ref('l10n_in.tax_tag_cess')

        def filter_invl_to_apply(invoice_line):
            return bool(invoice_line.product_id.l10n_in_hsn_code)

        def grouping_key_generator(base_line, _tax_values):
            # The rate is only for SGST/CGST.
            if base_line['is_refund']:
                tax_rep_field = 'refund_repartition_line_ids'
            else:
                tax_rep_field = 'invoice_repartition_line_ids'
            gst_taxes = base_line['taxes'].flatten_taxes_hierarchy()[tax_rep_field]\
                .filtered(lambda tax_rep: (
                    tax_rep.repartition_type == 'tax'
                    and any(tag in tax_rep.tag_ids for tag in tag_sgst + tag_cgst + tag_igst)
                ))\
                .tax_id

            return {
                'l10n_in_hsn_code': base_line['record'].product_id.l10n_in_hsn_code,
                'rate': sum(gst_taxes.mapped('amount')),
                'uom': base_line['record'].product_uom_id,
            }

        aggregated_values = self._prepare_invoice_aggregated_taxes(
            filter_invl_to_apply=filter_invl_to_apply,
            grouping_key_generator=grouping_key_generator,
        )

        results_map = {}
        has_igst = False
        has_gst = False
        has_cess = False
        for grouping_key, tax_details in aggregated_values['tax_details'].items():
            values = results_map.setdefault(grouping_key, {
                'quantity': 0.0,
                'amount_untaxed': tax_details['base_amount_currency'],
                'tax_amounts': defaultdict(lambda: 0.0),
            })

            # Quantity.
            invoice_line_ids = set()
            for invoice_line in tax_details['records']:
                if invoice_line.id not in invoice_line_ids:
                    values['quantity'] += invoice_line.quantity
                    invoice_line_ids.add(invoice_line.id)

            # Tax amounts.
            for tax_details in tax_details['group_tax_details']:
                tax_rep = tax_details['tax_repartition_line']
                if tag_igst in tax_rep.tag_ids:
                    has_igst = True
                    values['tax_amounts'][tag_igst] += tax_details['tax_amount_currency']
                if tag_cgst in tax_rep.tag_ids:
                    has_gst = True
                    values['tax_amounts'][tag_cgst] += tax_details['tax_amount_currency']
                if tag_sgst in tax_rep.tag_ids:
                    has_gst = True
                    values['tax_amounts'][tag_sgst] += tax_details['tax_amount_currency']
                if tag_cess in tax_rep.tag_ids:
                    has_cess = True
                    values['tax_amounts'][tag_cess] += tax_details['tax_amount_currency']

        # In case of base_line with HSN code but no taxes, an entry in results_map should be created.
        for base_line, _to_update_vals, _tax_values_list in aggregated_values['to_process']:
            if base_line['taxes']:
                continue

            grouping_key = frozendict(grouping_key_generator(base_line, None))
            results = results_map.setdefault(grouping_key, {
                'quantity': 0.0,
                'amount_untaxed': 0.0,
                'tax_amounts': defaultdict(lambda: 0.0),
            })
            results['quantity'] += base_line['quantity']
            results['amount_untaxed'] += base_line['price_subtotal']

        nb_columns = 5
        if has_igst:
            nb_columns += 1
        if has_gst:
            nb_columns += 2
        if has_cess:
            nb_columns += 1

        items = []
        for grouping_key, values in results_map.items():
            items.append({
                'l10n_in_hsn_code': grouping_key['l10n_in_hsn_code'],
                'quantity': values['quantity'],
                'uom': grouping_key['uom'],
                'rate': grouping_key['rate'],
                'amount_untaxed': values['amount_untaxed'],
                'tax_amount_igst': values['tax_amounts'].get(tag_igst, 0.0),
                'tax_amount_cgst': values['tax_amounts'].get(tag_cgst, 0.0),
                'tax_amount_sgst': values['tax_amounts'].get(tag_sgst, 0.0),
                'tax_amount_cess': values['tax_amounts'].get(tag_cess, 0.0),
            })

        return {
            'has_igst': has_igst,
            'has_gst': has_gst,
            'has_cess': has_cess,
            'nb_columns': nb_columns,
            'display_uom': display_uom,
            'items': items,
        }

```

## File: models\account_tax.py

```python
from odoo import api, fields, models


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

## File: models\company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models

class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_in_upi_id = fields.Char(string="UPI Id")
    l10n_in_gst_state_warning = fields.Char(related="partner_id.l10n_in_gst_state_warning")

    def create(self, vals):
        res = super().create(vals)
        # Update Fiscal Positions for new branch
        res._update_l10n_in_fiscal_position()
        return res

    def write(self, vals):
        res = super().write(vals)
        if (vals.get('state_id') or vals.get('country_id')) and not self.env.context.get('delay_account_group_sync'):
            # Update Fiscal Positions for companies setting up state for the first time
            self._update_l10n_in_fiscal_position()
        return res

    def _update_l10n_in_fiscal_position(self):
        companies_need_update_fp = self.filtered(lambda c: c.parent_ids[0].chart_template == 'in')
        for company in companies_need_update_fp:
            ChartTemplate = self.env['account.chart.template'].with_company(company)
            fiscal_position_data = ChartTemplate._get_in_account_fiscal_position()
            ChartTemplate._load_data({'account.fiscal.position': fiscal_position_data})

    def action_update_state_as_per_gstin(self):
        self.ensure_one()
        self.partner_id.action_update_state_as_per_gstin()

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

    l10n_in_audit_log_preview = fields.Html(string="Audit Preview", compute="_compute_l10n_in_audit_log_preview")
    l10n_in_audit_log_account_move_id = fields.Many2one('account.move', string="Accounting Entry", compute="_compute_l10n_in_audit_log_document_name", search="_search_l10n_in_audit_log_document_name")

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
            trackings = [
                (
                    fmt_vals['changedField'],
                    fmt_vals['oldValue']['value'],
                    fmt_vals['newValue']['value'],
                ) for fmt_vals in tracking_value_ids._tracking_value_format()
            ]
            for field_desc, old_value, new_value in trackings:
                audit_log_preview += Markup(
                    "<li>%(old_value)s <i class='o_TrackingValue_separator fa fa-long-arrow-right mx-1 text-600' title='%(title)s' role='img' aria-label='%(title)s'></i>%(new_value)s (%(field)s)</li>"
                ) % {
                    'old_value': old_value,
                    'new_value': new_value,
                    'title': _("Changed"),
                    'field': field_desc,
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
    module_l10n_in_edi = fields.Boolean('Indian Electronic Invoicing')
    module_l10n_in_edi_ewaybill = fields.Boolean('Indian Electronic Waybill')

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

    l10n_in_pan = fields.Char(
        string="PAN",
        help="PAN enables the department to link all transactions of the person with the department.\n"
             "These transactions include taxpayments, TDS/TCS credits, returns of income/wealth/gift/FBT,"
             " specified transactions, correspondence, and so on.\n"
             "Thus, PAN acts as an identifier for the person with the tax department."
    )

    display_pan_warning = fields.Boolean(string="Display pan warning", compute="_compute_display_pan_warning")
    l10n_in_gst_state_warning = fields.Char(compute="_compute_l10n_in_gst_state_warning")

    @api.depends('vat', 'state_id', 'country_id', 'fiscal_country_codes')
    def _compute_l10n_in_gst_state_warning(self):
        for partner in self:
            if (
                "IN" in partner.fiscal_country_codes
                and partner.check_vat_in(partner.vat)
            ):
                if partner.vat[:2] == "99":
                    partner.l10n_in_gst_state_warning = _(
                        "As per GSTN the country should be other than India, so it's recommended to"
                    )
                else:
                    state_id = self.env['res.country.state'].search([('l10n_in_tin', '=', partner.vat[:2])], limit=1)
                    if state_id and state_id != partner.state_id:
                        partner.l10n_in_gst_state_warning = _(
                            "As per GSTN the state should be %s, so it's recommended to", state_id.name
                        )
                    else:
                        partner.l10n_in_gst_state_warning = False
            else:
                partner.l10n_in_gst_state_warning = False

    @api.depends('l10n_in_pan')
    def _compute_display_pan_warning(self):
        self.display_pan_warning = self.vat and self.l10n_in_pan and self.l10n_in_pan != self.vat[2:12]

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
            if self.vat[2].isalpha():
                self.l10n_in_pan = self.vat[2:12]

    @api.model
    def _commercial_fields(self):
        res = super()._commercial_fields()
        return res + ['l10n_in_gst_treatment', 'l10n_in_pan']

    def check_vat_in(self, vat):
        """
            This TEST_GST_NUMBER is used as test credentials for EDI
            but this is not a valid number as per the regular expression
            so TEST_GST_NUMBER is considered always valid
        """
        if vat == TEST_GST_NUMBER:
            return True
        return super().check_vat_in(vat)

    def action_update_state_as_per_gstin(self):
        self.ensure_one()
        state_id = self.env['res.country.state'].search([('l10n_in_tin', '=', self.vat[:2])], limit=1)
        self.state_id = state_id
        if self.ref_company_ids:
            self.ref_company_ids._update_l10n_in_fiscal_position()

```

## File: models\template_in.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, fields, _
from odoo.addons.account.models.chart_template import template
from odoo import Command


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('in')
    def _get_in_template_data(self):
        return {
            'property_account_receivable_id': 'p10040',
            'property_account_payable_id': 'p11211',
            'property_account_expense_categ_id': 'p2107',
            'property_account_income_categ_id': 'p20011',
            'code_digits': '6',
            'display_invoice_amount_total_words': True,
        }

    @template('in', 'res.company')
    def _get_in_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.in',
                'bank_account_code_prefix': '1002',
                'cash_account_code_prefix': '1001',
                'transfer_account_code_prefix': '1008',
                'account_default_pos_receivable_account_id': 'p10041',
                'income_currency_exchange_account_id': 'p2013',
                'expense_currency_exchange_account_id': 'p2117',
                'account_journal_early_pay_discount_loss_account_id': 'p2132',
                'account_journal_early_pay_discount_gain_account_id': '2012',
                'account_opening_date': fields.Date.context_today(self).replace(month=4, day=1),
                'fiscalyear_last_month': '3',
                'account_sale_tax_id': 'sgst_sale_5',
                'account_purchase_tax_id': 'sgst_purchase_5',
            },
        }

    @template('in', 'account.fiscal.position')
    def _get_in_account_fiscal_position(self):
        company = self.env.company
        state_ids = [Command.set(company.state_id.ids)] if company.state_id else False
        intra_state_name = company.state_id and _('Within %s', company.state_id.name) or _('Intra State')
        state_specific = {
            'fiscal_position_in_intra_state': {
                'name': intra_state_name,
                'sequence': 1,
                'auto_apply': True,
                'state_ids': state_ids,
                'country_id': self.env.ref('base.in').id,
            },
            'fiscal_position_in_inter_state': {
                'name': _('Inter State'),
                'sequence': 2,
                'auto_apply': True,
                'tax_ids': self._get_l10n_in_fiscal_tax_vals(),
                'country_id': self.env.ref('base.in').id
            },
        }
        if company.parent_id:
            return state_specific
        return {
            **state_specific,
            'fiscal_position_in_export_sez_in': {
                'name': _('Export/SEZ'),
                'sequence': 3,
                'auto_apply': True,
                'tax_ids': self._get_l10n_in_fiscal_tax_vals(),
            },
            'fiscal_position_in_lut_sez': {
                'name': _('LUT - Export/SEZ'),
                'sequence': 4,
                'tax_ids': self._get_l10n_in_fiscal_tax_vals(use_zero_rated_igst=True),
            },
        }

    def _get_l10n_in_fiscal_tax_vals(self, use_zero_rated_igst=False):
        return [Command.clear()] + [
            Command.create({
                'tax_src_id': f"sgst_{tax_type}_{rate}",
                'tax_dest_id': f"igst_{tax_type}_{rate if not use_zero_rated_igst else 0}",
            })
            for tax_type in ["sale", "purchase"]
            for rate in [1, 2, 5, 12, 18, 28]  # Available existing GST Rates
        ]

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_in
from . import account_tax
from . import account_invoice
from . import company
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
                <field name="res_id" column_invisible="True"/>
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
                <filter string="Update Only" name="update_only" domain="[('tracking_value_ids', '!=', False)]" groups="base.group_system"/>
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

## File: static\src\components\hsn_autocomplete\hsn_autocomplete.js

```javascript
/** @odoo-module **/

import { AutoComplete } from "@web/core/autocomplete/autocomplete";
import { useChildRef } from "@web/core/utils/hooks";
import { registry } from "@web/core/registry";
import { _t } from "@web/core/l10n/translation";
import { CharField, charField } from "@web/views/fields/char/char_field";
import { useInputField } from "@web/views/fields/input_field_hook";

const l10N_IN_HSN_SERVICE_URL = "https://services.gst.gov.in/commonservices/hsn/search/qsearch";

export class L10nInHsnAutoComplete extends CharField {
    static template = "l10n_in.hsnAutoComplete";
    static components = {
        ...CharField.components,
        AutoComplete,
    };
    static props = {
        ...CharField.props,
        l10nInHsnDescription: { type: String, optional: true },
    };

    setup() {
        super.setup();
        this.inputRef = useChildRef();
        useInputField({
            getValue: () => this.props.record.data[this.props.name] || "",
            parse: (v) => this.parse(v),
            ref: this.inputRef,
        });
    }

    async getHsnSuggestions(value) {
        const suggestions = [];
        const onlyDigits = !isNaN(value) && value.indexOf(" ") < 0;
        const params = [
            { type: "byCode", category: "null" }, // For code
            { type: "byDesc", category: "P" }, // For products
            { type: "byDesc", category: "S" }, // For services
        ];
        const filteredParams = onlyDigits ? [params[0]] : params.slice(1);
        try {
            await Promise.all(
                filteredParams.map(async (param) => {
                    const controller = new AbortController();
                    const signal = controller.signal;
                    setTimeout(() => controller.abort(), 5000);
                    const res = await fetch(
                        `${l10N_IN_HSN_SERVICE_URL}?inputText=${value}&selectedType=${param.type}&category=${param.category}`,
                        { signal }
                    );
                    if (!res.ok) {
                        throw new Error(res.statusText);
                    }
                    const resData = await res.json();
                    if (resData.data) {
                        suggestions.push(
                            ...resData.data
                                .filter((item) => item.c.length > 3)
                                .map((item) => ({
                                    label: item.c,
                                    description: item.n,
                                }))
                        );
                    }
                })
            );
        } catch (e) {
            suggestions.push({
                label: _t("Could not contact API"),
                unselectable: false,
            });
            console.warn("HSN Autocomplete API error:", e);
        }
        return suggestions;
    }

    get sources() {
        return [
            {
                options: async (request) => {
                    if (request?.length > 2) {
                        return await this.getHsnSuggestions(request);
                    } else {
                        return [];
                    }
                },
                optionTemplate: "hsn_autocomplete.DropdownOption",
                placeholder: _t("Searching..."),
            },
        ];
    }

    onSelect(option) {
        const data = { [this.props.name]: option.label };
        if (this.props.l10nInHsnDescription) {
            data[this.props.l10nInHsnDescription] = option.description;
        }
        this.props.record.update(data);
    }
}

export const l10nInHsnAutoComplete = {
    ...charField,
    component: L10nInHsnAutoComplete,
    supportedOptions: [
        {
            label: _t("hsn description field"),
            name: "hsn_description_field",
            type: "string",
        },
    ],
    extractProps: ({ options }) => ({
        l10nInHsnDescription: options.hsn_description_field,
    }),
};

registry.category("fields").add("l10n_in_hsn_autocomplete", l10nInHsnAutoComplete);

```

## File: static\src\components\hsn_autocomplete\hsn_autocomplete.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<templates>
    <t t-name="l10n_in.hsnAutoComplete">
        <AutoComplete
            value="props.record.data[props.name] || ''"
            sources="sources"
            onSelect.bind="onSelect"
            input="inputRef"
            placeholder="props.placeholder || ''"
        />
    </t>

    <t t-name="hsn_autocomplete.DropdownOption">
        <div class="text-wrap">
            <strong t-out="option.label"/>
            <div t-out="option.description"/>
        </div>
    </t>
</templates>

```

## File: static\src\img\Google_Pay-Logo.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" height="800" width="1200" class="main-header__logo-image" fill="#A1A1A1" viewBox="-65.39955 -43.28375 566.7961 259.7025"><path fill="#5f6368" d="M206.197 84.585v50.75h-16.1V10.005h42.7a38.61 38.61 0 0127.65 10.85 34.88 34.88 0 0111.55 26.45 34.72 34.72 0 01-11.55 26.6q-11.2 10.68-27.65 10.67h-26.6zm0-59.15v43.75h27a21.28 21.28 0 0015.93-6.48 21.36 21.36 0 000-30.63 21 21 0 00-15.93-6.65h-27zm102.9 21.35q17.85 0 28.18 9.54 10.33 9.54 10.32 26.16v52.85h-15.4v-11.9h-.7q-10 14.7-26.6 14.7-14.17 0-23.71-8.4a26.82 26.82 0 01-9.54-21q0-13.31 10.06-21.17 10.06-7.86 26.86-7.88 14.34 0 23.62 5.25v-3.68a18.33 18.33 0 00-6.65-14.25 22.8 22.8 0 00-15.54-5.87q-13.49 0-21.35 11.38l-14.18-8.93q11.7-16.8 34.63-16.8zm-20.83 62.3a12.86 12.86 0 005.34 10.5 19.64 19.64 0 0012.51 4.2 25.67 25.67 0 0018.11-7.52q8-7.53 8-17.67-7.53-6-21-6-9.81 0-16.36 4.73c-4.41 3.2-6.6 7.09-6.6 11.76zm147.73-59.5l-53.76 123.55h-16.62l19.95-43.23-35.35-80.32h17.5l25.55 61.6h.35l24.85-61.6z"/><path fill="#4285f4" d="M141.137 73.645a85.79 85.79 0 00-1.24-14.64h-67.9v27.73h38.89a33.33 33.33 0 01-14.38 21.88v18h23.21c13.59-12.53 21.42-31.06 21.42-52.97z"/><path fill="#34a853" d="M71.997 144.005c19.43 0 35.79-6.38 47.72-17.38l-23.21-18c-6.46 4.38-14.78 6.88-24.51 6.88-18.78 0-34.72-12.66-40.42-29.72H7.667v18.55a72 72 0 0064.33 39.67z"/><path fill="#fbbc04" d="M31.577 85.785a43.14 43.14 0 010-27.56v-18.55H7.667a72 72 0 000 64.66z"/><path fill="#ea4335" d="M71.997 28.505a39.09 39.09 0 0127.62 10.8l20.55-20.55A69.18 69.18 0 0071.997.005a72 72 0 00-64.33 39.67l23.91 18.55c5.7-17.06 21.64-29.72 40.42-29.72z"/></svg>
```

## File: static\src\img\Paytm-Logo.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" height="800" width="1200" viewBox="-2.5242255 -1.318595 21.876621 7.91157"><g fill="none"><path d="M16.77137 1.55572c-.15275-.43674-.56903-.75036-1.05798-.75036h-.01023c-.31785 0-.6043.13229-.80821.34466-.20426-.21237-.49072-.34466-.80822-.34466h-.01023c-.2794 0-.5348.1023-.73095.27164V.99092c-.0067-.08573-.07726-.1531-.1644-.1531h-.75c-.09172 0-.1658.07408-.1658.16615v4.07282c0 .09207.07408.16616.1658.16616h.75c.08361 0 .1524-.06244.16334-.14323l-.00035-2.92382a.26293.26293 0 01.0014-.02928c.012-.13053.1076-.23777.2586-.25118h.13827c.06315.00564.11642.02787.1584.06138.06526.05186.1016.13159.1016.21908l.00282 2.90936c0 .09207.07443.1665.1658.1665h.75c.08856 0 .16052-.07055.16476-.15839l-.00035-2.92135c-.00036-.09596.0441-.18274.12206-.23425.03845-.0247.08467-.04127.13793-.04621h.13829c.16228.01411.26035.13723.26.28046l.00282 2.90548c0 .09207.07444.16615.1658.16615h.75001c.09137 0 .1658-.07408.1658-.16615V1.95259c0-.21308-.02398-.30374-.05679-.39687M11.69398.84843h-.42897V.15134C11.265.06773 11.19727 0 11.11366 0c-.00988 0-.0194.0014-.02857.00317-.47555.13053-.3803.78917-1.24848.84526h-.08432c-.0127 0-.02469.00176-.03633.00423h-.0007l.0007.00035c-.07409.01659-.12982.0822-.12982.16123v.75c0 .09137.07443.1658.16615.1658h.45262l-.0007 3.1803c0 .09066.07337.16404.16403.16404h.74154c.09031 0 .1637-.07338.1637-.16404l.00034-3.1803h.42016c.09137 0 .1658-.07443.1658-.1658v-.75c0-.09137-.07443-.1658-.1658-.1658" fill="#00BAF2"/><path d="M8.99555.84843h-.75c-.09137 0-.16546.07444-.16546.1658v1.55082c-.00176.09595-.07937.17286-.17568.17286h-.31397c-.09737 0-.17604-.07832-.17604-.17569l-.00282-1.54798c0-.09137-.07444-.1658-.1658-.1658h-.75001c-.09172 0-.1658.07443-.1658.1658v1.69968c0 .64558.46037 1.10596 1.1063 1.10596 0 0 .48472 0 .49954.00282.08749.00988.15557.08325.15557.17356 0 .08926-.06667.16228-.1531.17322-.00423.0007-.00811.00176-.0127.00247l-1.09679.00388c-.09172 0-.1658.07443-.1658.1658v.74966c0 .09172.07408.1658.1658.1658h1.22626c.64628 0 1.1063-.46002 1.1063-1.10595v-3.1369c0-.09137-.07408-.1658-.1658-.1658M1.73602 2.22268v.46285c0 .097-.07867.17603-.17568.17603l-.4759.00035v-.92745h.4759c.09701 0 .17568.07831.17568.17568zM1.80199.84825H.16263C.07267.84825 0 .92128 0 1.01088v.73484c0 .00141.00035.00282.00035.00423 0 .00353-.00035.00706-.00035.01023v3.32599c0 .09031.06773.16405.1517.16616h.7641c.09137 0 .1658-.07408.1658-.1658l.00283-1.13983h.71755c.60042 0 1.01882-.41663 1.01882-1.01953V1.8692c0-.6029-.4184-1.02094-1.01882-1.02094zM4.8479 3.98346v.11712c0 .00953-.0014.0187-.00281.02752a.17498.17498 0 01-.00706.02434c-.02329.06562-.0889.11324-.16687.11324h-.3122c-.09737 0-.17674-.07408-.17674-.1651v-.14146c0-.00176-.00036-.00353-.00036-.00529l.00036-.37641v-.11784l.00035-.00105c.00035-.09067.07902-.16404.17639-.16404h.3122c.09772 0 .17675.07373.17675.1651zM4.72868.85256h-1.0407c-.09207 0-.1665.06985-.1665.15557v.29175c0 .00176.00035.00388.00035.00564 0 .00212-.00036.00423-.00036.00635v.3997c0 .09067.07903.16475.1764.16475h.99094c.07832.01235.14041.0695.14923.15875v.09666c-.00882.08502-.0702.1471-.145.15416h-.4907c-.65264 0-1.1176.43357-1.1176 1.04246v.87207c0 .60537.3997 1.0361 1.04774 1.0361h1.35996c.24412 0 .44203-.18485.44203-.41239V1.97827c0-.69003-.3556-1.12571-1.2058-1.12571z" fill="#1F336B"/></g></svg>
```

## File: static\src\img\PhonePe-Logo.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" height="800" width="1200" xml:space="preserve" y="0" x="0" id="Layer_2" version="1.1" viewBox="-19.77372 -8.949675 171.37224 53.69805"><style id="style28" type="text/css">.st0{fill:#5f259f}</style><g transform="translate(.0248 -6.071)" id="g34"><circle r="17.9" id="ellipse30" cy="24" cx="17.9" class="st0" transform="matrix(.2298 -.9732 .9732 .2298 -9.5957 35.8754)"/><path id="path32" d="M90.5 34.2v-6.5c0-1.6-.6-2.4-2.1-2.4-.6 0-1.3.1-1.7.2V35c0 .3-.3.6-.6.6h-2.3c-.3 0-.6-.3-.6-.6V23.9c0-.4.3-.7.6-.8 1.5-.5 3-.8 4.6-.8 3.6 0 5.6 1.9 5.6 5.4v7.4c0 .3-.3.6-.6.6H92c-.9 0-1.5-.7-1.5-1.5zm9-3.9l-.1.9c0 1.2.8 1.9 2.1 1.9 1 0 1.9-.3 2.9-.8.1 0 .2-.1.3-.1.2 0 .3.1.4.2.1.1.3.4.3.4.2.3.4.7.4 1 0 .5-.3 1-.7 1.2-1.1.6-2.4.9-3.8.9-1.6 0-2.9-.4-3.9-1.2-1-.9-1.6-2.1-1.6-3.6v-3.9c0-3.1 2-5 5.4-5 3.3 0 5.2 1.8 5.2 5v2.4c0 .3-.3.6-.6.6h-6.3zm-.1-2.2h3.8v-1c0-1.2-.7-2-1.9-2s-1.9.7-1.9 2zm25.5 2.2l-.1.9c0 1.2.8 1.9 2.1 1.9 1 0 1.9-.3 2.9-.8.1 0 .2-.1.3-.1.2 0 .3.1.4.2.1.1.3.4.3.4.2.3.4.7.4 1 0 .5-.3 1-.7 1.2-1.1.6-2.4.9-3.8.9-1.6 0-2.9-.4-3.9-1.2-1-.9-1.6-2.1-1.6-3.6v-3.9c0-3.1 2-5 5.4-5 3.3 0 5.2 1.8 5.2 5v2.4c0 .3-.3.6-.6.6h-6.3zm-.1-2.2h3.8v-1c0-1.2-.7-2-1.9-2s-1.9.7-1.9 2zM66 35.7h1.4c.3 0 .6-.3.6-.6v-7.4c0-3.4-1.8-5.4-4.8-5.4-.9 0-1.9.2-2.5.4V19c0-.8-.7-1.5-1.5-1.5h-1.4c-.3 0-.6.3-.6.6v17c0 .3.3.6.6.6h2.3c.3 0 .6-.3.6-.6v-9.4c.5-.2 1.2-.3 1.7-.3 1.5 0 2.1.7 2.1 2.4v6.5c.1.7.7 1.4 1.5 1.4zm15.1-8.4V31c0 3.1-2.1 5-5.6 5-3.4 0-5.6-1.9-5.6-5v-3.7c0-3.1 2.1-5 5.6-5 3.5 0 5.6 1.9 5.6 5zm-3.5 0c0-1.2-.7-2-2-2s-2 .7-2 2V31c0 1.2.7 1.9 2 1.9s2-.7 2-1.9zm-22.3-1.7c0 3.2-2.4 5.4-5.6 5.4-.8 0-1.5-.1-2.2-.4v4.5c0 .3-.3.6-.6.6h-2.3c-.3 0-.6-.3-.6-.6V19.2c0-.4.3-.7.6-.8 1.5-.5 3-.8 4.6-.8 3.6 0 6.1 2.2 6.1 5.6zM51.7 23c0-1.6-1.1-2.4-2.6-2.4-.9 0-1.5.3-1.5.3v6.6c.6.3.9.4 1.6.4 1.5 0 2.6-.9 2.6-2.4V23zm68.2 2.6c0 3.2-2.4 5.4-5.6 5.4-.8 0-1.5-.1-2.2-.4v4.5c0 .3-.3.6-.6.6h-2.3c-.3 0-.6-.3-.6-.6V19.2c0-.4.3-.7.6-.8 1.5-.5 3-.8 4.6-.8 3.6 0 6.1 2.2 6.1 5.6zm-3.6-2.6c0-1.6-1.1-2.4-2.6-2.4-.9 0-1.5.3-1.5.3v6.6c.6.3.9.4 1.6.4 1.5 0 2.6-.9 2.6-2.4V23z" class="st0"/></g><path id="path36" d="M26.0248 13.229c0-.7-.6-1.3-1.3-1.3h-2.4l-5.5-6.3c-.5-.6-1.3-.8-2.1-.6l-1.9.6c-.3.1-.4.5-.2.7l6 5.7h-9.1c-.3 0-.5.2-.5.5v1c0 .7.6 1.3 1.3 1.3h1.4v4.8c0 3.6 1.9 5.7 5.1 5.7 1 0 1.8-.1 2.8-.5v3.2c0 .9.7 1.6 1.6 1.6h1.4c.3 0 .6-.3.6-.6v-14.3h2.3c.3 0 .5-.2.5-.5zm-6.4 8.6c-.6.3-1.4.4-2 .4-1.6 0-2.4-.8-2.4-2.6v-4.8h4.4z" fill="#fff"/></svg>
```

## File: static\src\img\Upi-logo.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="120" height="60" fill-rule="evenodd"><path d="M95.678 42.9L110 29.835l-6.784-13.516z" fill="#097939"/><path d="M90.854 42.9l14.322-13.065-6.784-13.516z" fill="#ed752e"/><path d="M22.41 16.47l-6.03 21.475 21.407.15 5.88-21.625h5.427l-7.05 25.14c-.27.96-1.298 1.74-2.295 1.74H12.31c-1.664 0-2.65-1.3-2.2-2.9l6.724-23.98zm66.182-.15h5.427l-7.538 27.03h-5.58zM49.698 27.582l27.136-.15 1.81-5.707H51.054l1.658-5.256 29.4-.27c1.83-.017 2.92 1.4 2.438 3.167L81.78 29.49c-.483 1.766-2.36 3.197-4.19 3.197H53.316L50.454 43.8h-5.28z" fill="#747474"/></svg>
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
                <field name="l10n_in_state_id" domain="[('country_id.code', '=', 'IN')]"
                    options="{'no_create': True, 'no_open': True}"
                    invisible="country_code != 'IN' or move_type == 'entry'"
                    readonly="state != 'draft'"
                    required="country_code == 'IN' and move_type != 'entry' and l10n_in_journal_type in ('sale', 'purchase')"/>
                <field name="l10n_in_gst_treatment"
                    invisible="country_code != 'IN' or move_type == 'entry'"
                    readonly="state != 'draft'"
                    required="country_code == 'IN' and move_type != 'entry'"/>
            </xpath>
            <xpath expr="//page[@id='other_tab']/group[@id='other_tab_group']" position="after">
                <group string="Export India" invisible="l10n_in_gst_treatment not in ['overseas', 'deemed_export'] or move_type not in ['out_invoice', 'out_refund']">
                    <field name="l10n_in_shipping_bill_number" readonly="state != 'draft'"/>
                    <field name="l10n_in_shipping_bill_date" readonly="state != 'draft'"/>
                    <field name="l10n_in_shipping_port_code_id" readonly="state != 'draft'"/>
                </group>
                <group string="Import India" invisible="l10n_in_gst_treatment not in ['overseas', 'special_economic_zone'] or move_type not in ['in_invoice', 'in_refund']">
                    <field name="l10n_in_shipping_bill_number" string="Bill of Entry Number" readonly="state != 'draft'"/>
                    <field name="l10n_in_shipping_bill_date" string="Bill of Entry Date" readonly="state != 'draft'"/>
                    <field name="l10n_in_shipping_port_code_id" readonly="state != 'draft'"/>
                </group>
            </xpath>
            <xpath expr="//field[@name='partner_shipping_id']" position="before">
                <field name="l10n_in_reseller_partner_id"
                       groups="l10n_in.group_l10n_in_reseller"
                       invisible="move_type not in ('out_invoice', 'out_refund') or country_code != 'IN' or move_type == 'entry'"
                       readonly="state != 'draft'"/>
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
            <field name="profit_account_id" position="attributes">
                <attribute name="invisible">country_code != 'IN' and type != 'cash' or country_code == 'IN' and type not in ['bank', 'cash', 'sale', 'purchase']</attribute>
            </field>
            <field name="loss_account_id" position="attributes">
                <attribute name="invisible">country_code != 'IN' and type != 'cash' or country_code == 'IN' and type not in ['bank', 'cash', 'sale', 'purchase']</attribute>
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
                <field name="l10n_in_reverse_charge" invisible="amount_type == 'group' or country_code != 'IN'"/>
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
                <field name="l10n_in_hsn_code" widget="l10n_in_hsn_autocomplete" invisible="'IN' not in fiscal_country_codes"/>
                <field name="l10n_in_hsn_description" invisible="1"/>
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

        <xpath expr="//div[@id='qrcode_info']" position="attributes">
            <attribute name="t-if" add="and o.company_id.account_fiscal_country_id.code != 'IN'" separator=" "/>
        </xpath>

        <xpath expr="//div[@id='qrcode_info']" position="after">
            <t t-if="o.company_id.account_fiscal_country_id.code == 'IN' and o.company_id.l10n_in_upi_id">
                <div style="display:-webkit-flex;" class="flex-column">
                    <strong>PAYMENT QR CODE</strong>
                    <div class="mt-1 mb-1">
                        <p class="mb-0">UPI ID:</p>
                        <span class="mb-0" t-field="o.company_id.l10n_in_upi_id"/>
                    </div>
                    <div class="d-flex flex-row" t-attf-style="#{'-webkit-transform:translateX(-0.5rem);' if report_type != 'html' else '-webkit-transform:translate(-0.5rem,-0.8rem);'}">
                        <img src="/l10n_in/static/src/img/PhonePe-Logo.svg" style="width:4.5rem;"/>
                        <img src="/l10n_in/static/src/img/Google_Pay-Logo.svg" style="width:3.5rem;"/>
                        <img src="/l10n_in/static/src/img/Paytm-Logo.svg" style="width:4rem;"/>
                        <img src="/l10n_in/static/src/img/Upi-logo.svg" t-attf-style="#{'' if report_type != 'html' else 'padding:0.5rem;'} width:4rem;"/>
                    </div>
                </div>
            </t>
        </xpath>

        <xpath expr="//h2" position="replace">
            <h2>
                <span t-if="o.move_type == 'out_invoice' and o.state == 'posted'" t-field="o.journal_id.name"/>
                <span t-elif="o.move_type == 'out_invoice' and o.state == 'draft'">Draft <span t-field="o.journal_id.name"/></span>
                <span t-elif="o.move_type == 'out_invoice' and o.state == 'cancel'">Cancelled <span t-field="o.journal_id.name"/></span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'posted'">Credit Note</span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'draft'">Draft Credit Note</span>
                <span t-elif="o.move_type == 'out_refund' and o.state == 'cancel'">Cancelled Credit Note</span>
                <span t-elif="o.move_type == 'in_refund'">Vendor Credit Note</span>
                <span t-elif="o.move_type == 'in_invoice'">Vendor Bill</span>
                <span t-if="o.name != '/'" t-field="o.name"/>
            </h2>
        </xpath>

        <xpath expr="//div[@id='payment_term']" position="before">
            <t t-if="o.company_id.account_fiscal_country_id.code == 'IN'">
                <t t-set="hsn_summary" t-value="o._l10n_in_get_hsn_summary_table()"/>
                <t t-if="hsn_summary">
                    <div name="l10n_in_hsn_summary" class="mt-3">
                        <table class="table table-sm table-borderless col-6" style="page-break-inside: avoid;">
                            <thead>
                                <th t-att-colspan="hsn_summary['nb_columns']"><h3>HSN Summary</h3></th>
                            </thead>
                            <thead>
                                <th>HSN/SAC</th>
                                <th class="text-end">Quantity</th>
                                <th class="text-end">Rate %</th>
                                <th class="text-end">Taxable Value</th>
                                <th class="text-end" t-if="hsn_summary['has_gst']">SGST</th>
                                <th class="text-end" t-if="hsn_summary['has_gst']">CGST</th>
                                <th class="text-end" t-if="hsn_summary['has_igst']">IGST</th>
                                <th class="text-end" t-if="hsn_summary['has_cess']">CESS</th>
                            </thead>
                            <tr t-foreach="hsn_summary['items']" t-as="item">
                                <td t-esc="item['l10n_in_hsn_code']"/>
                                <td class="text-end">
                                    <span t-esc="item['quantity']"/>
                                    <span t-if="hsn_summary['display_uom']">(<t t-esc="item['uom'].name"/>)</span>
                                </td>
                                <td class="text-end" t-esc="item['rate']"/>
                                <td class="text-end">
                                    <span t-esc="item['amount_untaxed']"
                                          t-options="{'widget': 'monetary', 'display_currency': o.currency_id}"/>
                                </td>
                                <td class="text-end" t-if="hsn_summary['has_gst']">
                                    <span t-esc="item['tax_amount_sgst']"
                                          t-options="{'widget': 'monetary', 'display_currency': o.currency_id}"/>
                                </td>
                                <td class="text-end" t-if="hsn_summary['has_gst']">
                                    <span t-esc="item['tax_amount_cgst']"
                                          t-options="{'widget': 'monetary', 'display_currency': o.currency_id}"/>
                                </td>
                                <td class="text-end" t-if="hsn_summary['has_igst']">
                                    <span t-esc="item['tax_amount_igst']"
                                          t-options="{'widget': 'monetary', 'display_currency': o.currency_id}"/>
                                </td>
                                <td class="text-end" t-if="hsn_summary['has_cess']">
                                    <span t-esc="item['tax_amount_cess']"
                                          t-options="{'widget': 'monetary', 'display_currency': o.currency_id}"/>
                                </td>
                            </tr>
                        </table>
                    </div>
                </t>
            </t>
        </xpath>

    </template>

    <!-- Workaround for Studio reports, see odoo/odoo#60660 -->
    <template id="report_invoice" inherit_id="account.report_invoice">
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

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_company_form" model="ir.ui.view">
        <field name="name">res.company.form.inherit.l10n_in_upi</field>
        <field name="model">res.company</field>
        <field name="inherit_id" ref="base.view_company_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='currency_id']" position="after">
                <field name="l10n_in_upi_id" invisible="country_code != 'IN'"/>
            </xpath>
            <xpath expr="//sheet" position="before">
                <div class="alert alert-warning mt-1 mb-1" role="alert" invisible="not l10n_in_gst_state_warning or country_code != 'IN'">
                    <field name="l10n_in_gst_state_warning"/>
                    <a name="action_update_state_as_per_gstin"
                            string="update it"
                            class="ms-1"
                            invisible="country_code != 'IN'"
                            type="object"/>
                </div>
            </xpath>
        </field>
    </record>
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
            <block id="invoicing_settings" position="inside">
                <setting help="Use this if setup with Reseller(E-Commerce)." name="ecommerce_reseller_setting" title="Manage Reseller(E-Commerce)" invisible="country_code != 'IN'">
                    <field name="group_l10n_in_reseller"/>
                </setting>
                <setting help="Connect to NIC (National Informatics Center) to submit invoices on posting." name="electronic_invoices_in" invisible="country_code != 'IN'" documentation="/applications/finance/accounting/fiscal_localizations/localizations/india.html#indian-e-invoicing">
                    <field name="module_l10n_in_edi" class="oe_inline"/>
                </setting>
                <setting help="Connect to NIC (National Informatics Center) to submit e-waybill on posting." name="electronic_waybill_in" invisible="country_code != 'IN'" documentation="/applications/finance/accounting/fiscal_localizations/localizations/india.html#indian-e-waybill">
                    <field name="module_l10n_in_edi_ewaybill" class="oe_inline"/>
                </setting>
            </block>
            <xpath expr="//app[@name='account']" position="inside">
                <div id="india_integration_section" invisible="1">
                    <block title="Indian Integration" id="india_localization" invisible="country_code != 'IN'">
                    </block>
                </div>
            </xpath>
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
                <field name="l10n_in_tin" invisible="country_id != %(base.in)d"/>
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
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="attributes">
                <attribute name="readonly">parent_id</attribute>
                <attribute name="required">l10n_in_gst_treatment in ['regular', 'composition', 'special_economic_zone', 'deemed_export']</attribute>
            </xpath>
            <xpath expr="//field[@name='vat']" position="before">
                <field name="l10n_in_gst_treatment" invisible="'IN' not in fiscal_country_codes" readonly="parent_id"/>
            </xpath>
            <xpath expr="//field[@name='vat']" position="after">
                <field name="l10n_in_pan" placeholder="e.g. ABCTY1234D" invisible="'IN' not in fiscal_country_codes" readonly="parent_id"/>
            </xpath>
            <xpath expr="//sheet" position="before">
                <div class="alert alert-warning mt-1 mb-1" role="alert" invisible="not l10n_in_gst_state_warning or country_code != 'IN'">
                    <field name="l10n_in_gst_state_warning"/>
                    <a name="action_update_state_as_per_gstin"
                            string="update it"
                            class="ms-1"
                            invisible="country_code != 'IN'"
                            type="object"/>
                </div>
                <field name="display_pan_warning" invisible="1"/>
                <div class="alert alert-warning" role="alert"
                        invisible="not display_pan_warning">
                        PAN number is not same as the 3rd to 12th characters of the GST number.
                </div>
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
                <field name="l10n_in_code" invisible="'IN' not in fiscal_country_codes"/>
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

