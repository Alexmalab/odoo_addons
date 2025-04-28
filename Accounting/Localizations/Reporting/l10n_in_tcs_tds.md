# Odoo Module: l10n_in_tcs_tds

Category: Accounting/Localizations/Reporting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

from odoo import api, SUPERUSER_ID

import logging

_logger = logging.getLogger(__name__)


def load_taxes(env):
    in_chart_template = env.ref('l10n_in.indian_chart_template_standard')
    for company in env['res.company'].search([('chart_template_id', '=', in_chart_template.id)]):
        try:
            with env.cr.savepoint():
                tax_template_ids = env['ir.model.data'].search([
                    ('module', '=', 'l10n_in_tcs_tds'),
                    ('model', '=', 'account.tax.template'),
                    ]).mapped('res_id')
                generated_tax_res = env['account.tax.template'].browse(tax_template_ids)._generate_tax(company)
                taxes_ref = generated_tax_res['tax_template_to_tax']
        except Exception:
            taxes_ref = {}
            _logger.error("Can't load TDS and TCS taxes for company: %s(%s).", company.name, company.id)
        if taxes_ref:
            try:
                with env.cr.savepoint():
                    account_ref = {}
                    # Generating Accounts from templates.
                    account_template_ref = in_chart_template.generate_account(taxes_ref, {}, in_chart_template.code_digits, company)
                    account_ref.update(account_template_ref)

                    # writing account values after creation of accounts
                    for key, value in generated_tax_res['account_dict']['account.tax.repartition.line'].items():
                        if value['account_id']:
                            key.write({
                                'account_id': account_ref.get(value['account_id']),
                            })
            except Exception:
                _logger.error("Can't load TCS and TDS account so account is not set in taxes of company: %s(%s).", company.name, company.id)


def l10n_in_post_init(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    load_taxes(env)

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': "Indian - TCS/TDS Accounting Report and Taxes",
    'version': '1.0',
    'description': """
Tax Report TCS/TDS for India
====================================

This module adds TCS and TDS Tax Report and load related Taxes in Indian Company.
    """,
    'category': 'Accounting/Localizations/Reporting',
    'depends': ['l10n_in'],
    'data': [
        'data/account_tax_group_data.xml',
        'data/account.account.template.csv',
        'data/account_tax_report_tcs_data.xml',
        'data/account_tax_template_tcs_data.xml',
        'data/account_tax_report_tds_data.xml',
        'data/account_tax_template_tds_data.xml',
        'views/res_partner_views.xml',
    ],
    'demo': [
    ],
    'post_init_hook': 'l10n_in_post_init',
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","account_type","chart_template_id/id","tag_ids/id","reconcile"
"p11244","TDS Deducted","11244","liability_current","l10n_in.indian_chart_template_standard","","False"
"p11245","TCS Collected","11245","liability_current","l10n_in.indian_chart_template_standard","","False"

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tcs_group" model="account.tax.group">
            <field name="name">TCS</field>
            <field name="country_id" ref="base.in"/>
            <field name="sequence">100</field>
        </record>
        <record id="tds_group" model="account.tax.group">
            <field name="name">TDS</field>
            <field name="country_id" ref="base.in"/>
            <field name="sequence">100</field>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_tcs_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
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
                <field name="sequence" eval="1"/>
            </record>
        </field>
        <field name="line_ids">
            <record id="tcs_report_line_section_206c_1_alfhc" model="account.report.line">
                <field name="name">Section 206C(1): Alcoholic Liquor for human consumption</field>
                <field name="sequence" eval="1"/>
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
                <field name="sequence" eval="2"/>
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
                <field name="sequence" eval="3"/>
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
                <field name="sequence" eval="4"/>
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
                <field name="sequence" eval="5"/>
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
                <field name="sequence" eval="6"/>
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
                <field name="sequence" eval="7"/>
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
                <field name="sequence" eval="8"/>
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
                <field name="sequence" eval="9"/>
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
                <field name="sequence" eval="10"/>
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
                <field name="sequence" eval="11"/>
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
                <field name="sequence" eval="12"/>
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
                <field name="sequence" eval="13"/>
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
                <field name="sequence" eval="14"/>
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
<odoo>
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
                <field name="sequence" eval="1"/>
            </record>
        </field>
        <field name="line_ids">
            <record id="tds_report_line_section_192" model="account.report.line">
                <field name="name">Section 192: Payment of salary</field>
                <field name="sequence" eval="1"/>
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
                <field name="sequence" eval="2"/>
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
                <field name="sequence" eval="3"/>
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
                <field name="sequence" eval="4"/>
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
                <field name="sequence" eval="5"/>
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
                <field name="sequence" eval="6"/>
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
                <field name="sequence" eval="7"/>
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
                <field name="sequence" eval="8"/>
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
                <field name="sequence" eval="9"/>
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
                <field name="sequence" eval="10"/>
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
                <field name="sequence" eval="11"/>
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
                <field name="sequence" eval="12"/>
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
                <field name="sequence" eval="13"/>
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
                <field name="sequence" eval="14"/>
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
                <field name="sequence" eval="15"/>
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
                <field name="sequence" eval="16"/>
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
                <field name="sequence" eval="17"/>
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
                <field name="sequence" eval="18"/>
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
                <field name="sequence" eval="19"/>
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
                <field name="sequence" eval="20"/>
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
                <field name="sequence" eval="21"/>
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
                <field name="sequence" eval="22"/>
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
                <field name="sequence" eval="23"/>
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
                <field name="sequence" eval="24"/>
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
                <field name="sequence" eval="25"/>
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
                <field name="sequence" eval="26"/>
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
                <field name="sequence" eval="27"/>
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
                <field name="sequence" eval="28"/>
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
                <field name="sequence" eval="29"/>
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
                <field name="sequence" eval="30"/>
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
                <field name="sequence" eval="31"/>
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

## File: data\account_tax_template_tcs_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tcs_1_us_206c_1_alfhc" model="account.tax.template">
        <field name="name">TCS @1% u/s 206C(1): Alcoholic Liquor for human consumption
    </field>
        <field name="description">TCS @1% u/s 206C(1): Alcoholic Liquor for human consumption
    </field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1_alfhc_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1_alfhc_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_5_us_206c_1_alfhc" model="account.tax.template">
        <field name="name">TCS @5% u/s 206C(1): Alcoholic Liquor for human consumption
</field>
        <field name="description">TCS @5% u/s 206C(1): Alcoholic Liquor for human consumption
</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1_alfhc_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1_alfhc_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_5_us_206c_1_tl" model="account.tax.template">
        <field name="name">TCS @5% u/s 206C(1): Tendu leaves</field>
        <field name="description">TCS @5% u/s 206C(1): Tendu leaves</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1_tl_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1_tl_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_2_5_us_206c_1_touafl" model="account.tax.template">
        <field name="name">TCS @2.5% u/s 206C(1): Timber obtained under a forest lease</field>
        <field name="description">TCS @2.5% u/s 206C(1): Timber obtained under a forest lease</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">2.5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1_touafl_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1_touafl_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_5_us_206c_1_touafl" model="account.tax.template">
        <field name="name">TCS @5% u/s 206C(1): Timber obtained under a forest lease</field>
        <field name="description">TCS @5% u/s 206C(1): Timber obtained under a forest lease</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1_touafl_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1_touafl_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_2_5_us_206c_1_tobamotuafl" model="account.tax.template">
        <field name="name">TCS @2.5% u/s 206C(1): Timber obtained by any mode other than under a forest lease</field>
        <field name="description">TCS @2.5% u/s 206C(1): Timber obtained by any mode other than under a forest lease</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">2.5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1_tobaotuafl_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1_tobaotuafl_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_5_us_206c_1_tobamotuafl" model="account.tax.template">
        <field name="name">TCS @5% u/s 206C(1): Timber obtained by any mode other than under a forest lease</field>
        <field name="description">TCS @5% u/s 206C(1): Timber obtained by any mode other than under a forest lease</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1_tobaotuafl_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1_tobaotuafl_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_2_5_us_206c_1_aofpnbtotl" model="account.tax.template">
        <field name="name">TCS @2.5% u/s 206C(1): Any other forest produce not being timber or tendu leaves</field>
        <field name="description">TCS @2.5% u/s 206C(1): Any other forest produce not being timber or tendu leaves</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">2.5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1_aofpnbtotl_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1_aofpnbtotl_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_5_us_206c_1_aofpnbtotl" model="account.tax.template">
        <field name="name">TCS @5% u/s 206C(1): Any other forest produce not being timber or tendu leaves</field>
        <field name="description">TCS @5% u/s 206C(1): Any other forest produce not being timber or tendu leaves</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1_aofpnbtotl_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1_aofpnbtotl_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_1_us_206c_1_s" model="account.tax.template">
        <field name="name">TCS @1% u/s 206C(1): Scrap</field>
        <field name="description">TCS @1% u/s 206C(1): Scrap</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1_s_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1_s_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_5_us_206c_1_s" model="account.tax.template">
        <field name="name">TCS @5% u/s 206C(1): Scrap</field>
        <field name="description">TCS @5% u/s 206C(1): Scrap</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1_s_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1_s_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_1_us_206c_1_mbcoloio" model="account.tax.template">
        <field name="name">TCS @1% u/s 206C(1): Minrals, being coal or lignite or iron ore</field>
        <field name="description">TCS @1% u/s 206C(1): Minrals, being coal or lignite or iron ore</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1_mbcoloio_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1_mbcoloio_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_5_us_206c_1_mbcoloio" model="account.tax.template">
        <field name="name">TCS @5% u/s 206C(1): Minrals, being coal or lignite or iron ore</field>
        <field name="description">TCS @5% u/s 206C(1): Minrals, being coal or lignite or iron ore</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1_mbcoloio_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1_mbcoloio_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_2_us_206c_1c_pl" model="account.tax.template">
        <field name="name">TCS @2% u/s 206C(1C): Parking lot</field>
        <field name="description">TCS @2% u/s 206C(1C): Parking lot</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">2</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1c_pl_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1c_pl_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_5_us_206c_1c_pl" model="account.tax.template">
        <field name="name">TCS @5% u/s 206C(1C): Parking lot</field>
        <field name="description">TCS @5% u/s 206C(1C): Parking lot</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1c_pl_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1c_pl_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_2_us_206c_1c_tp" model="account.tax.template">
        <field name="name">TCS @2% u/s 206C(1C): Toll plaza</field>
        <field name="description">TCS @2% u/s 206C(1C): Toll plaza</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">2</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1c_tp_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1c_tp_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_5_us_206c_1c_tp" model="account.tax.template">
        <field name="name">TCS @5% u/s 206C(1C): Toll plaza</field>
        <field name="description">TCS @5% u/s 206C(1C): Toll plaza</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1c_tp_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1c_tp_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_2_us_206c_1c_maq" model="account.tax.template">
        <field name="name">TCS @2% u/s 206C(1C): Mining and quarrying</field>
        <field name="description">TCS @2% u/s 206C(1C): Mining and quarrying</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">2</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1c_maq_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1c_maq_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_5_us_206c_1c_maq" model="account.tax.template">
        <field name="name">TCS @5% u/s 206C(1C): Mining and quarrying</field>
        <field name="description">TCS @5% u/s 206C(1C): Mining and quarrying</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1c_maq_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1c_maq_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_1_us_206c_1f_mv" model="account.tax.template">
        <field name="name">TCS @1% u/s 206C(1F): Motor Vehicle</field>
        <field name="description">TCS @1% u/s 206C(1F): Motor Vehicle</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1f_mv_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1f_mv_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_5_us_206c_1f_mv" model="account.tax.template">
        <field name="name">TCS @5% u/s 206C(1F): Motor Vehicle</field>
        <field name="description">TCS @5% u/s 206C(1F): Motor Vehicle</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1f_mv_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1f_mv_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_5_us_206c_1g_som" model="account.tax.template">
        <field name="name">TCS @5% u/s 206C(1G): Sum of money (above 7 lakhs) for remittance out of India</field>
        <field name="description">TCS @5% u/s 206C(1G): Sum of money (above 7 lakhs) for remittance out of India</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1g_som_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1g_som_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_5_us_206c_1g_soaotpp" model="account.tax.template">
        <field name="name">TCS @5% u/s 206C(1G): Seller of an overseas tour program package</field>
        <field name="description">TCS @5% u/s 206C(1G): Seller of an overseas tour program package</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1g_soaotpp_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1g_soaotpp_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_0_1_us_206c_1h_sog" model="account.tax.template">
        <field name="name">TCS @0.1% u/s 206C(1H): Sale of Goods</field>
        <field name="description">TCS @0.1% u/s 206C(1H): Sale of Goods</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">0.1</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1h_sog_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1h_sog_tag')],
            }),
        ]"/>
    </record>
    <record id="tcs_1_us_206c_1h_sog" model="account.tax.template">
        <field name="name">TCS @1% u/s 206C(1H): Sale of Goods</field>
        <field name="description">TCS @1% u/s 206C(1H): Sale of Goods</field>
        <field name="type_tax_use">sale</field>
        <field name="amount_type">percent</field>
        <field name="amount">1</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tcs_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11245'),
                'plus_report_expression_ids': [ref('tcs_report_line_section_206c_1h_sog_tag')],
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
                'account_id': ref('p11245'),
                'minus_report_expression_ids': [ref('tcs_report_line_section_206c_1h_sog_tag')],
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\account_tax_template_tds_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tds_10_us_192a" model="account.tax.template">
        <field name="name">TDS @10% u/s 192A</field>
        <field name="description">TDS @10% u/s 192A</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_192a_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_192a_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_192a" model="account.tax.template">
        <field name="name">TDS @20% u/s 192A</field>
        <field name="description">TDS @20% u/s 192A</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_192a_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_192a_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_10_us_193" model="account.tax.template">
        <field name="name">TDS @10% u/s 193</field>
        <field name="description">TDS @10% u/s 193</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_193_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_193_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_193" model="account.tax.template">
        <field name="name">TDS @20% u/s 193</field>
        <field name="description">TDS @20% u/s 193</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_193_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_193_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_10_us_194" model="account.tax.template">
        <field name="name">TDS @10% u/s 194</field>
        <field name="description">TDS @10% u/s 194</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194" model="account.tax.template">
        <field name="name">TDS @20% u/s 194</field>
        <field name="description">TDS @20% u/s 194</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_10_us_194a" model="account.tax.template">
        <field name="name">TDS @10% u/s 194A</field>
        <field name="description">TDS @10% u/s 194A</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194a_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194a_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194a" model="account.tax.template">
        <field name="name">TDS @20% u/s 194A</field>
        <field name="description">TDS @20% u/s 194A</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194a_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194a_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_30_us_194b" model="account.tax.template">
        <field name="name">TDS @30% u/s 194B</field>
        <field name="description">TDS @30% u/s 194B</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-30</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194b_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194b_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_30_us_194bb" model="account.tax.template">
        <field name="name">TDS @30% u/s 194BB</field>
        <field name="description">TDS @30% u/s 194BB</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-30</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194bb_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194bb_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_1_us_194c" model="account.tax.template">
        <field name="name">TDS @1% u/s 194C</field>
        <field name="description">TDS @1% u/s 194C</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-1</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194c_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194c_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_2_us_194c" model="account.tax.template">
        <field name="name">TDS @2% u/s 194C</field>
        <field name="description">TDS @2% u/s 194C</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-2</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194c_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194c_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194c" model="account.tax.template">
        <field name="name">TDS @20% u/s 194C</field>
        <field name="description">TDS @20% u/s 194C</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194c_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194c_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_5_us_194d" model="account.tax.template">
        <field name="name">TDS @5% u/s 194D</field>
        <field name="description">TDS @5% u/s 194D</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-5</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194d_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194d_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_10_us_194d" model="account.tax.template">
        <field name="name">TDS @10% u/s 194D</field>
        <field name="description">TDS @10% u/s 194D</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194d_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194d_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194d" model="account.tax.template">
        <field name="name">TDS @20% u/s 194D</field>
        <field name="description">TDS @20% u/s 194D</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194d_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194d_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_5_us_194da" model="account.tax.template">
        <field name="name">TDS @5% u/s 194DA</field>
        <field name="description">TDS @5% u/s 194DA</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-5</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194da_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194da_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194da" model="account.tax.template">
        <field name="name">TDS @20% u/s 194DA</field>
        <field name="description">TDS @20% u/s 194DA</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194da_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194da_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194e" model="account.tax.template">
        <field name="name">TDS @20% u/s 194E</field>
        <field name="description">TDS @20% u/s 194E</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194e_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194e_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_10_us_194ee" model="account.tax.template">
        <field name="name">TDS @10% u/s 194EE</field>
        <field name="description">TDS @10% u/s 194EE</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194ee_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194ee_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194ee" model="account.tax.template">
        <field name="name">TDS @20% u/s 194EE</field>
        <field name="description">TDS @20% u/s 194EE</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194ee_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194ee_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194f" model="account.tax.template">
        <field name="name">TDS @20% u/s 194F</field>
        <field name="description">TDS @20% u/s 194F</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194f_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194f_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_5_us_194g" model="account.tax.template">
        <field name="name">TDS @5% u/s 194G</field>
        <field name="description">TDS @5% u/s 194G</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-5</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194g_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194g_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194g" model="account.tax.template">
        <field name="name">TDS @20% u/s 194G</field>
        <field name="description">TDS @20% u/s 194G</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194g_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194g_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_5_us_194h" model="account.tax.template">
        <field name="name">TDS @5% u/s 194H</field>
        <field name="description">TDS @5% u/s 194H</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-5</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194h_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194h_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194h" model="account.tax.template">
        <field name="name">TDS @20% u/s 194H</field>
        <field name="description">TDS @20% u/s 194H</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194h_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194h_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_2_us_194i" model="account.tax.template">
        <field name="name">TDS @2% u/s 194I</field>
        <field name="description">TDS @2% u/s 194I</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-2</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194i_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194i_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_10_us_194i" model="account.tax.template">
        <field name="name">TDS @10% u/s 194I</field>
        <field name="description">TDS @10% u/s 194I</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194i_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194i_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194i" model="account.tax.template">
        <field name="name">TDS @20% u/s 194I</field>
        <field name="description">TDS @20% u/s 194I</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194i_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194i_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_1_us_194ia" model="account.tax.template">
        <field name="name">TDS @1% u/s 194IA</field>
        <field name="description">TDS @1% u/s 194IA</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-1</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194ia_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194ia_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194ia" model="account.tax.template">
        <field name="name">TDS @20% u/s 194IA</field>
        <field name="description">TDS @20% u/s 194IA</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194ia_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194ia_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_5_us_194ib" model="account.tax.template">
        <field name="name">TDS @5% u/s 194IB</field>
        <field name="description">TDS @5% u/s 194IB</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-5</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194ib_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194ib_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194ib" model="account.tax.template">
        <field name="name">TDS @20% u/s 194IB</field>
        <field name="description">TDS @20% u/s 194IB</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194ib_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194ib_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_10_us_194ic" model="account.tax.template">
        <field name="name">TDS @10% u/s 194IC</field>
        <field name="description">TDS @10% u/s 194IC</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194ic_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194ic_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194ic" model="account.tax.template">
        <field name="name">TDS @20% u/s 194IC</field>
        <field name="description">TDS @20% u/s 194IC</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194ic_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194ic_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_2_us_194j" model="account.tax.template">
        <field name="name">TDS @2% u/s 194J</field>
        <field name="description">TDS @2% u/s 194J</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-2</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194j_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194j_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_10_us_194j" model="account.tax.template">
        <field name="name">TDS @10% u/s 194J</field>
        <field name="description">TDS @10% u/s 194J</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194j_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194j_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194j" model="account.tax.template">
        <field name="name">TDS @20% u/s 194J</field>
        <field name="description">TDS @20% u/s 194J</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194j_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194j_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_10_us_194k" model="account.tax.template">
        <field name="name">TDS @10% u/s 194K</field>
        <field name="description">TDS @10% u/s 194K</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194k_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194k_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194k" model="account.tax.template">
        <field name="name">TDS @20% u/s 194K</field>
        <field name="description">TDS @20% u/s 194K</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194k_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194k_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_10_us_194la" model="account.tax.template">
        <field name="name">TDS @10% u/s 194LA</field>
        <field name="description">TDS @10% u/s 194LA</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194la_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194la_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194la" model="account.tax.template">
        <field name="name">TDS @20% u/s 194LA</field>
        <field name="description">TDS @20% u/s 194LA</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194la_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194la_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_10_us_194lba" model="account.tax.template">
        <field name="name">TDS @10% u/s 194LBA(1)</field>
        <field name="description">TDS @10% u/s 194LBA(1)</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194lba_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194lba_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194lba" model="account.tax.template">
        <field name="name">TDS @20% u/s 194LBA(1)</field>
        <field name="description">TDS @20% u/s 194LBA(1)</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194lba_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194lba_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_10_us_194lbb" model="account.tax.template">
        <field name="name">TDS @10% u/s 194LBB</field>
        <field name="description">TDS @10% u/s 194LBB</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194lbb_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194lbb_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194lbb" model="account.tax.template">
        <field name="name">TDS @20% u/s 194LBB</field>
        <field name="description">TDS @20% u/s 194LBB</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194lbb_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194lbb_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_5_us_194lb" model="account.tax.template">
        <field name="name">TDS @5% u/s 194LB</field>
        <field name="description">TDS @5% u/s 194LB</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-5</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194lb_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194lb_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194lb" model="account.tax.template">
        <field name="name">TDS @20% u/s 194LB</field>
        <field name="description">TDS @20% u/s 194LB</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194lb_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194lb_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_10_us_194lbc" model="account.tax.template">
        <field name="name">TDS @10% u/s 194LBC</field>
        <field name="description">TDS @10% u/s 194LBC</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194lbc_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194lbc_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_25_us_194lbc" model="account.tax.template">
        <field name="name">TDS @25% u/s 194LBC</field>
        <field name="description">TDS @25% u/s 194LBC</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-25</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194lbc_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194lbc_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_30_us_194lbc" model="account.tax.template">
        <field name="name">TDS @30% u/s 194LBC</field>
        <field name="description">TDS @30% u/s 194LBC</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-30</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194lbc_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194lbc_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_5_us_194m" model="account.tax.template">
        <field name="name">TDS @5% u/s 194M</field>
        <field name="description">TDS @5% u/s 194M</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-5</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194m_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194m_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194m" model="account.tax.template">
        <field name="name">TDS @20% u/s 194M</field>
        <field name="description">TDS @20% u/s 194M</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194m_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194m_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_2_us_194n" model="account.tax.template">
        <field name="name">TDS @2% u/s 194N</field>
        <field name="description">TDS @2% u/s 194N</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-2</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194n_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194n_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_5_us_194n" model="account.tax.template">
        <field name="name">TDS @5% u/s 194N</field>
        <field name="description">TDS @5% u/s 194N</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-5</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194n_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194n_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194n" model="account.tax.template">
        <field name="name">TDS @20% u/s 194N</field>
        <field name="description">TDS @20% u/s 194N</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194n_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194n_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_1_us_194o" model="account.tax.template">
        <field name="name">TDS @1% u/s 194O</field>
        <field name="description">TDS @1% u/s 194O</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-1</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194o_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194o_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_us_194o" model="account.tax.template">
        <field name="name">TDS @20% u/s 194O</field>
        <field name="description">TDS @20% u/s 194O</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194o_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194o_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_01_us_194q" model="account.tax.template">
        <field name="name">TDS @0.1% u/s 194Q</field>
        <field name="description">TDS @0.1% u/s 194Q</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="amount_type">percent</field>
        <field name="amount">-0.1</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_194q_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_194q_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_10_4_us_195" model="account.tax.template">
        <field name="name">TDS @10.4% u/s 195</field>
        <field name="description">TDS @10.4% u/s 195</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-10.4</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_195_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_195_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_15_6_us_195" model="account.tax.template">
        <field name="name">TDS @15.6% u/s 195</field>
        <field name="description">TDS @15.6% u/s 195</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-15.6</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_195_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_195_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_20_8_us_195" model="account.tax.template">
        <field name="name">TDS @20.8% u/s 195</field>
        <field name="description">TDS @20.8% u/s 195</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-20.8</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_195_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_195_tag')],
            }),
        ]"/>
    </record>
    <record id="tds_31_2_us_195" model="account.tax.template">
        <field name="name">TDS @31.2% u/s 195</field>
        <field name="description">TDS @31.2% u/s 195</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="amount_type">percent</field>
        <field name="amount">-31.2</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tds_group"/>
        <field name="chart_template_id" ref="l10n_in.indian_chart_template_standard"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('p11244'),
                'minus_report_expression_ids': [ref('tds_report_line_section_195_tag')],
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
                'account_id': ref('p11244'),
                'plus_report_expression_ids': [ref('tds_report_line_section_195_tag')],
            }),
        ]"/>
    </record>
</odoo>

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_in_pan = fields.Char(string="PAN", help="""PAN enables the department to link all transactions of the person with the department.
                                These transactions include taxpayments, TDS/TCS credits, returns of income/wealth/gift/FBT, specified transactions, correspondence, and so on.
                                thus, PAN acts as an identifier for the person with the tax department.""")

    @api.model
    def _commercial_fields(self):
        res = super()._commercial_fields()
        return res + ['l10n_in_pan']

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_in_tcs_tds_view_partner_form" model="ir.ui.view">
        <field name="name">l10n.in.tcs.tds.res.partner.vat.inherit</field>
        <field name="model">res.partner</field>
        <field name="priority" eval="90" />
        <field name="inherit_id" ref="base.view_partner_form" />
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="after">
                <field name="l10n_in_pan" placeholder="e.g. ABCTY1234D" />
            </xpath>
        </field>
    </record>
</odoo>

```

