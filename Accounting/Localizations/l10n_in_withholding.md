# Odoo Module: l10n_in_withholding

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models
from . import wizard

import logging

from odoo.exceptions import ValidationError

_logger = logging.getLogger(__name__)


def _l10n_in_withholding_post_init(env):
    """ Existing companies that have the Indian Chart of Accounts set """
    for company in env['res.company'].search([('chart_template', '=', 'in'), ('parent_id', '=', False)]):
        _logger.info("Company %s already has the Indian localization installed, updating...", company.name)
        ChartTemplate = env['account.chart.template'].with_company(company)
        data = {
            model: ChartTemplate._parse_csv('in', model, module='l10n_in_withholding')
            for model in [
                'account.account',
                'account.tax',
            ]
        }
        try:
            ChartTemplate._deref_account_tags('in', data['account.tax'])
            ChartTemplate._pre_reload_data(company, {}, data)
            ChartTemplate._load_data(data)
            company.l10n_in_withholding_account_id = ChartTemplate.ref('p100595')
        except ValidationError as e:
            _logger.warning("Error while updating Chart of Accounts for company %s: %s", company.name, e.args[0])
        tds_group_id = ChartTemplate.ref("tds_group", raise_if_not_found=False)
        if tds_group_id:
            tds_purchase_taxes = env['account.tax'].with_context(active_test=False).search([('tax_group_id', '=', tds_group_id.id), ('type_tax_use', '=', 'purchase')])
            tds_purchase_taxes.write({'l10n_in_tds_tax_type': 'purchase', 'type_tax_use': 'none'})

```

## File: __manifest__.py

```python
{
    'name': 'Indian - TDS and TCS',
    'version': '1.0',
    'countries': ['in'],
    'description': """
        Support for Indian TDS (Tax Deducted at Source).
    """,
    'category': 'Accounting/Localizations',
    'depends': ['l10n_in'],
    'data': [
        'security/ir.model.access.csv',
        'data/l10n_in.section.alert.csv',
        'data/account_tax_report_tcs_data.xml',
        'data/account_tax_report_tds_data.xml',
        'wizard/l10n_in_withhold_wizard.xml',
        'views/l10n_in_section_alert_views.xml',
        'views/account_account_views.xml',
        'views/account_move_views.xml',
        'views/account_move_line_views.xml',
        'views/account_payment_views.xml',
        'views/account_tax_views.xml',
        'views/res_config_settings_views.xml',
    ],
    'post_init_hook': '_l10n_in_withholding_post_init',
    'license': 'LGPL-3',
}

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

## File: data\l10n_in.section.alert.csv

```csv
"id","name","consider_amount","is_per_transaction_limit","per_transaction_limit","is_aggregate_limit","aggregate_limit","aggregate_period","tax_source_type"
"tds_section_192","192","untaxed_amount","","","","","fiscal_yearly","tds"
"tds_section_192a","192A","untaxed_amount","","","True","50000","fiscal_yearly","tds"
"tds_section_193","193","untaxed_amount","","","True","5000","fiscal_yearly","tds"
"tds_section_194","194","untaxed_amount","","","True","5000","fiscal_yearly","tds"
"tds_section_194a","194A","untaxed_amount","","","True","5000","fiscal_yearly","tds"
"tds_section_194b","194B","untaxed_amount","","","True","10000","fiscal_yearly","tds"
"tds_section_194ba","194BA","untaxed_amount","","","True","0","fiscal_yearly","tds"
"tds_section_194bb","194BB","untaxed_amount","","","True","10000","fiscal_yearly","tds"
"tds_section_194c","194C","untaxed_amount","True","30000","True","100000","fiscal_yearly","tds"
"tds_section_194d","194D","untaxed_amount","","","True","15000","fiscal_yearly","tds"
"tds_section_194da","194DA","untaxed_amount","","","True","100000","fiscal_yearly","tds"
"tds_section_194e","194E","untaxed_amount","","","True","0","fiscal_yearly","tds"
"tds_section_194ee","194EE","untaxed_amount","","","True","2500","fiscal_yearly","tds"
"tds_section_194f","194F","untaxed_amount","","","True","0","fiscal_yearly","tds"
"tds_section_194g","194G","untaxed_amount","","","True","15000","fiscal_yearly","tds"
"tds_section_194h","194H","untaxed_amount","","","True","15000","fiscal_yearly","tds"
"tds_section_194i","194I","untaxed_amount","","","True","240000","fiscal_yearly","tds"
"tds_section_194ia","194IA","untaxed_amount","True","5000000","","","fiscal_yearly","tds"
"tds_section_194ib","194IB","untaxed_amount","","","True","50000","monthly","tds"
"tds_section_194ic","194IC","untaxed_amount","","","True","0","fiscal_yearly","tds"
"tds_section_194j","194J","untaxed_amount","","","True","30000","fiscal_yearly","tds"
"tds_section_194j_dir","194J(DIRECTORS)","untaxed_amount","","","True","0","fiscal_yearly","tds"
"tds_section_194k","194K","untaxed_amount","","","True","5000","fiscal_yearly","tds"
"tds_section_194la","194LA","untaxed_amount","","","True","250000","fiscal_yearly","tds"
"tds_section_194lba1","194LBA(1)","untaxed_amount","","","True","0","fiscal_yearly","tds"
"tds_section_194lbb","194LBB","untaxed_amount","","","True","0","fiscal_yearly","tds"
"tds_section_194lb","194LB","untaxed_amount","","","True","0","fiscal_yearly","tds"
"tds_section_194lbc","194LBC","untaxed_amount","","","True","0","fiscal_yearly","tds"
"tds_section_194m","194M","untaxed_amount","","","True","5000000","fiscal_yearly","tds"
"tds_section_194n","194N","untaxed_amount","","","True","10000000","fiscal_yearly","tds"
"tds_section_194o_huf","194O(HUF)","untaxed_amount","","","True","500000","fiscal_yearly","tds"
"tds_section_194o","194O","untaxed_amount","","","True","0","fiscal_yearly","tds"
"tds_section_194q","194Q","untaxed_amount","","","True","5000000","fiscal_yearly","tds"
"tds_section_195","195","untaxed_amount","","","True","0","fiscal_yearly","tds"
"tcs_section_206c1_alc","206C(1) Liquor","total_amount","","","True","0","fiscal_yearly","tcs"
"tcs_section_206c1_tl","206C(1) Tendu leaves","total_amount","","","True","0","fiscal_yearly","tcs"
"tcs_section_206c1_tim","206C(1) Timber woods(FL)","total_amount","","","True","0","fiscal_yearly","tcs"
"tcs_section_206c1_tim_o","206C(1) Timber woods","total_amount","","","True","0","fiscal_yearly","tcs"
"tcs_section_206c1_fo","206C(1) OFP","total_amount","","","True","0","fiscal_yearly","tcs"
"tcs_section_206c1_sc","206C(1) Scrap","total_amount","","","True","0","fiscal_yearly","tcs"
"tcs_section_206c1_min","206C(1) Min","total_amount","","","True","0","fiscal_yearly","tcs"
"tcs_section_206c1c_p","206C(1C) Parking lot","total_amount","","","True","0","fiscal_yearly","tcs"
"tcs_section_206c1c_t","206C(1C) Toll plaza","total_amount","","","True","0","fiscal_yearly","tcs"
"tcs_section_206c1c_mq","206C(1C) MQ","total_amount","","","True","0","fiscal_yearly","tcs"
"tcs_section_206c1f_mv","206C(1F) Motor Vehicle","total_amount","True","1000000","","","fiscal_yearly","tcs"
"tcs_section_206c1g_r","206C(1G) Remittance","total_amount","","","True","700000","fiscal_yearly","tcs"
"tcs_section_206c1g_ot","206C(1G) Overseas Tour","total_amount","","","True","0","fiscal_yearly","tcs"
"tcs_section206c1h_g","206C(1H)","total_amount","","","True","5000000","fiscal_yearly","tcs"

```

## File: data\template\account.account-in.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","l10n_in_tds_tcs_section_id"
"p100595",TDS (Withholding Control),"100595","asset_current","","False",""
"p211210","Professional Fees","211210","expense","","False","l10n_in_withholding.tds_section_194j"
"p211220","Audit Fees","211220","expense","","False","l10n_in_withholding.tds_section_194j"
"p211230","Job Work Expense","211230","expense","","False","l10n_in_withholding.tds_section_194c"
"p211240","Advertisement Expense","211240","expense","","False","l10n_in_withholding.tds_section_194c"
"p211250","Commission/Brokerage Expense","211250","expense","","False","l10n_in_withholding.tds_section_194h"

```

## File: data\template\account.tax-in.csv

```csv
"id","name","description","invoice_label",type_tax_use,l10n_in_tds_tax_type,"amount_type","amount","tax_scope","active","tax_group_id","is_base_affected","include_base_amount","children_tax_ids","python_compute","sequence","l10n_in_reverse_charge","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","l10n_in_section_id"
"tds_sale_10_us_192a","10% TDS 192A S","TDS @10% u/s 192A","TDS @10% u/s 192A","none","sale","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_192a"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_192a","20% TDS 192A S","TDS @20% u/s 192A","TDS @20% u/s 192A","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_192a"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_10_us_193","10% TDS 193 S","TDS @10% u/s 193","TDS @10% u/s 193","none","sale","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_193"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_193","20% TDS 193 S","TDS @20% u/s 193","TDS @20% u/s 193","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_193"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_10_us_194","10% TDS 194 S","TDS @10% u/s 194","TDS @10% u/s 194","none","sale","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194","20% TDS 194 S","TDS @20% u/s 194","TDS @20% u/s 194","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_10_us_194a","10% TDS 194A S","TDS @10% u/s 194A","TDS @10% u/s 194A","none","sale","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194a"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194a","20% TDS 194A S","TDS @20% u/s 194A","TDS @20% u/s 194A","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194a"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_30_us_194b","30% TDS 194B S","TDS @30% u/s 194B","TDS @30% u/s 194B","none","sale","percent","-30.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194b"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_30_us_194bb","30% TDS 194BB S","TDS @30% u/s 194BB","TDS @30% u/s 194BB","none","sale","percent","-30.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194bb"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_1_us_194c","1% TDS 194C S","TDS @1% u/s 194C","TDS @1% u/s 194C","none","sale","percent","-1.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194c"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_2_us_194c","2% TDS 194C S","TDS @2% u/s 194C","TDS @2% u/s 194C","none","sale","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194c"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194c","20% TDS 194C S","TDS @20% u/s 194C","TDS @20% u/s 194C","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194c"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_5_us_194d","5% TDS 194D S","TDS @5% u/s 194D","TDS @5% u/s 194D","none","sale","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194d"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_10_us_194d","10% TDS 194D S","TDS @10% u/s 194D","TDS @10% u/s 194D","none","sale","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194d"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194d","20% TDS 194D S","TDS @20% u/s 194D","TDS @20% u/s 194D","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194d"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_2_us_194da","2% TDS 194DA S","TDS @2% u/s 194DA","TDS @2% u/s 194DA","none","sale","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194da"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_5_us_194da","5% TDS 194DA S","TDS @5% u/s 194DA","TDS @5% u/s 194DA","none","sale","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194da"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194da","20% TDS 194DA S","TDS @20% u/s 194DA","TDS @20% u/s 194DA","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194da"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194e","20% TDS 194E S","TDS @20% u/s 194E","TDS @20% u/s 194E","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194e"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_10_us_194ee","10% TDS 194EE S","TDS @10% u/s 194EE","TDS @10% u/s 194EE","none","sale","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ee"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194ee","20% TDS 194EE S","TDS @20% u/s 194EE","TDS @20% u/s 194EE","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ee"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194f","20% TDS 194F S","TDS @20% u/s 194F","TDS @20% u/s 194F","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194f"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_2_us_194g","2% TDS 194G S","TDS @2% u/s 194G","TDS @2% u/s 194G","none","sale","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194g"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_5_us_194g","5% TDS 194G S","TDS @5% u/s 194G","TDS @5% u/s 194G","none","sale","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194g"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194g","20% TDS 194G S","TDS @20% u/s 194G","TDS @20% u/s 194G","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194g"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_2_us_194h","2% TDS 194H S","TDS @2% u/s 194H","TDS @2% u/s 194H","none","sale","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194h"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_5_us_194h","5% TDS 194H S","TDS @5% u/s 194H","TDS @5% u/s 194H","none","sale","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194h"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194h","20% TDS 194H S","TDS @20% u/s 194H","TDS @20% u/s 194H","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194h"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_2_us_194i","2% TDS 194I S","TDS @2% u/s 194I","TDS @2% u/s 194I","none","sale","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194i"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_10_us_194i","10% TDS 194I S","TDS @10% u/s 194I","TDS @10% u/s 194I","none","sale","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194i"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194i","20% TDS 194I S","TDS @20% u/s 194I","TDS @20% u/s 194I","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194i"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_1_us_194ia","1% TDS 194IA S","TDS @1% u/s 194IA","TDS @1% u/s 194IA","none","sale","percent","-1.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ia"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194ia","20% TDS 194IA S","TDS @20% u/s 194IA","TDS @20% u/s 194IA","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ia"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_2_us_194ib","2% TDS 194IB S","TDS @2% u/s 194IB","TDS @2% u/s 194IB","none","sale","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ib"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_5_us_194ib","5% TDS 194IB S","TDS @5% u/s 194IB","TDS @5% u/s 194IB","none","sale","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ib"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194ib","20% TDS 194IB S","TDS @20% u/s 194IB","TDS @20% u/s 194IB","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ib"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_10_us_194ic","10% TDS 194IC S","TDS @10% u/s 194IC","TDS @10% u/s 194IC","none","sale","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ic"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194ic","20% TDS 194IC S","TDS @20% u/s 194IC","TDS @20% u/s 194IC","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ic"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_2_us_194j","2% TDS 194J S","TDS @2% u/s 194J","TDS @2% u/s 194J","none","sale","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194j"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_10_us_194j","10% TDS 194J S","TDS @10% u/s 194J","TDS @10% u/s 194J","none","sale","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194j"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194j","20% TDS 194J S","TDS @20% u/s 194J","TDS @20% u/s 194J","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194j"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_10_us_194k","10% TDS 194K S","TDS @10% u/s 194K","TDS @10% u/s 194K","none","sale","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194k"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194k","20% TDS 194K S","TDS @20% u/s 194K","TDS @20% u/s 194K","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194k"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_10_us_194la","10% TDS 194LA S","TDS @10% u/s 194LA","TDS @10% u/s 194LA","none","sale","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194la"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194la","20% TDS 194LA S","TDS @20% u/s 194LA","TDS @20% u/s 194LA","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194la"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_10_us_194lba","10% TDS 194LBA(1) S","TDS @10% u/s 194LBA(1)","TDS @10% u/s 194LBA(1)","none","sale","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lba1"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194lba","20% TDS 194LBA(1) S","TDS @20% u/s 194LBA(1)","TDS @20% u/s 194LBA(1)","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lba1"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_10_us_194lbb","10% TDS 194LBB S","TDS @10% u/s 194LBB","TDS @10% u/s 194LBB","none","sale","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lbb"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194lbb","20% TDS 194LBB S","TDS @20% u/s 194LBB","TDS @20% u/s 194LBB","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lbb"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_5_us_194lb","5% TDS 194LB S","TDS @5% u/s 194LB","TDS @5% u/s 194LB","none","sale","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lb"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194lb","20% TDS 194LB S","TDS @20% u/s 194LB","TDS @20% u/s 194LB","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lb"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_10_us_194lbc","10% TDS 194LBC S","TDS @10% u/s 194LBC","TDS @10% u/s 194LBC","none","sale","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lbc"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_25_us_194lbc","25% TDS 194LBC S","TDS @25% u/s 194LBC","TDS @25% u/s 194LBC","none","sale","percent","-25.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lbc"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_30_us_194lbc","30% TDS 194LBC S","TDS @30% u/s 194LBC","TDS @30% u/s 194LBC","none","sale","percent","-30.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lbc"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_2_us_194m","2% TDS 194M S","TDS @2% u/s 194M","TDS @2% u/s 194M","none","sale","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194m"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_5_us_194m","5% TDS 194M S","TDS @5% u/s 194M","TDS @5% u/s 194M","none","sale","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194m"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194m","20% TDS 194M S","TDS @20% u/s 194M","TDS @20% u/s 194M","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194m"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_2_us_194n","2% TDS 194N S","TDS @2% u/s 194N","TDS @2% u/s 194N","none","sale","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194n"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_5_us_194n","5% TDS 194N S","TDS @5% u/s 194N","TDS @5% u/s 194N","none","sale","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194n"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194n","20% TDS 194N S","TDS @20% u/s 194N","TDS @20% u/s 194N","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194n"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_01_us_194o","0.1% TDS 194O S","TDS @0.1% u/s 194O","TDS @0.1% u/s 194O","none","sale","percent","-0.1","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194o"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_1_us_194o","1% TDS 194O S","TDS @1% u/s 194O","TDS @1% u/s 194O","none","sale","percent","-1.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194o"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_us_194o","20% TDS 194O S","TDS @20% u/s 194O","TDS @20% u/s 194O","none","sale","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194o"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_01_us_194q","0.1% TDS 194Q S","TDS @0.1% u/s 194Q","TDS @0.1% u/s 194Q","none","sale","percent","-0.1","consu","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194q"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_10_4_us_195","10.4% TDS 195 S","TDS @10.4% u/s 195","TDS @10.4% u/s 195","none","sale","percent","-10.4","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_195"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_15_6_us_195","15.6% TDS 195 S","TDS @15.6% u/s 195","TDS @15.6% u/s 195","none","sale","percent","-15.6","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_195"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_20_8_us_195","20.8% TDS 195 S","TDS @20.8% u/s 195","TDS @20.8% u/s 195","none","sale","percent","-20.8","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_195"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_sale_31_2_us_195","31.2% TDS 195 S","TDS @31.2% u/s 195","TDS @31.2% u/s 195","none","sale","percent","-31.2","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_195"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p10058","",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p10058","",""
"tds_10_us_192a","10% TDS 192A P","TDS @10% u/s 192A","TDS @10% u/s 192A","none","purchase","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_192a"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-192A",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+192A",""
"tds_20_us_192a","20% TDS 192A P","TDS @20% u/s 192A","TDS @20% u/s 192A","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_192a"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-192A",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+192A",""
"tds_10_us_193","10% TDS 193 P","TDS @10% u/s 193","TDS @10% u/s 193","none","purchase","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_193"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-193",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+193",""
"tds_20_us_193","20% TDS 193 P","TDS @20% u/s 193","TDS @20% u/s 193","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_193"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-193",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+193",""
"tds_10_us_194","10% TDS 194 P","TDS @10% u/s 194","TDS @10% u/s 194","none","purchase","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194",""
"tds_20_us_194","20% TDS 194 P","TDS @20% u/s 194","TDS @20% u/s 194","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194",""
"tds_10_us_194a","10% TDS 194A P","TDS @10% u/s 194A","TDS @10% u/s 194A","none","purchase","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194a"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194A",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194A",""
"tds_20_us_194a","20% TDS 194A P","TDS @20% u/s 194A","TDS @20% u/s 194A","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194a"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194A",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194A",""
"tds_30_us_194b","30% TDS 194B P","TDS @30% u/s 194B","TDS @30% u/s 194B","none","purchase","percent","-30.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194b"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194B",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194B",""
"tds_30_us_194bb","30% TDS 194BB P","TDS @30% u/s 194BB","TDS @30% u/s 194BB","none","purchase","percent","-30.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194bb"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194BB",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194BB",""
"tds_1_us_194c","1% TDS 194C P","TDS @1% u/s 194C","TDS @1% u/s 194C","none","purchase","percent","-1.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194c"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194C",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194C",""
"tds_2_us_194c","2% TDS 194C P","TDS @2% u/s 194C","TDS @2% u/s 194C","none","purchase","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194c"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194C",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194C",""
"tds_20_us_194c","20% TDS 194C P","TDS @20% u/s 194C","TDS @20% u/s 194C","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194c"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194C",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194C",""
"tds_5_us_194d","5% TDS 194D P","TDS @5% u/s 194D","TDS @5% u/s 194D","none","purchase","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194d"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194D",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194D",""
"tds_10_us_194d","10% TDS 194D P","TDS @10% u/s 194D","TDS @10% u/s 194D","none","purchase","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194d"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194D",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194D",""
"tds_20_us_194d","20% TDS 194D P","TDS @20% u/s 194D","TDS @20% u/s 194D","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194d"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194D",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194D",""
"tds_2_us_194da","2% TDS 194DA P","TDS @2% u/s 194DA","TDS @2% u/s 194DA","none","purchase","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194da"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194DA",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194DA",""
"tds_5_us_194da","5% TDS 194DA P","TDS @5% u/s 194DA","TDS @5% u/s 194DA","none","purchase","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194da"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194DA",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194DA",""
"tds_20_us_194da","20% TDS 194DA P","TDS @20% u/s 194DA","TDS @20% u/s 194DA","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194da"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194DA",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194DA",""
"tds_20_us_194e","20% TDS 194E P","TDS @20% u/s 194E","TDS @20% u/s 194E","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194e"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194E",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194E",""
"tds_10_us_194ee","10% TDS 194EE P","TDS @10% u/s 194EE","TDS @10% u/s 194EE","none","purchase","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ee"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194EE",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194EE",""
"tds_20_us_194ee","20% TDS 194EE P","TDS @20% u/s 194EE","TDS @20% u/s 194EE","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ee"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194EE",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194EE",""
"tds_20_us_194f","20% TDS 194F P","TDS @20% u/s 194F","TDS @20% u/s 194F","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194f"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194F",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194F",""
"tds_2_us_194g","2% TDS 194G P","TDS @2% u/s 194G","TDS @2% u/s 194G","none","purchase","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194g"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194G",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194G",""
"tds_5_us_194g","5% TDS 194G P","TDS @5% u/s 194G","TDS @5% u/s 194G","none","purchase","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194g"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194G",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194G",""
"tds_20_us_194g","20% TDS 194G P","TDS @20% u/s 194G","TDS @20% u/s 194G","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194g"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194G",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194G",""
"tds_2_us_194h","2% TDS 194H P","TDS @2% u/s 194H","TDS @2% u/s 194H","none","purchase","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194h"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194H",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194H",""
"tds_5_us_194h","5% TDS 194H P","TDS @5% u/s 194H","TDS @5% u/s 194H","none","purchase","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194h"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194H",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194H",""
"tds_20_us_194h","20% TDS 194H P","TDS @20% u/s 194H","TDS @20% u/s 194H","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194h"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194H",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194H",""
"tds_2_us_194i","2% TDS 194I P","TDS @2% u/s 194I","TDS @2% u/s 194I","none","purchase","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194i"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194I",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194I",""
"tds_10_us_194i","10% TDS 194I P","TDS @10% u/s 194I","TDS @10% u/s 194I","none","purchase","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194i"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194I",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194I",""
"tds_20_us_194i","20% TDS 194I P","TDS @20% u/s 194I","TDS @20% u/s 194I","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194i"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194I",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194I",""
"tds_1_us_194ia","1% TDS 194IA P","TDS @1% u/s 194IA","TDS @1% u/s 194IA","none","purchase","percent","-1.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ia"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194IA",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194IA",""
"tds_20_us_194ia","20% TDS 194IA P","TDS @20% u/s 194IA","TDS @20% u/s 194IA","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ia"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194IA",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194IA",""
"tds_2_us_194ib","2% TDS 194IB P","TDS @2% u/s 194IB","TDS @2% u/s 194IB","none","purchase","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ib"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194IB",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194IB",""
"tds_5_us_194ib","5% TDS 194IB P","TDS @5% u/s 194IB","TDS @5% u/s 194IB","none","purchase","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ib"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194IB",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194IB",""
"tds_20_us_194ib","20% TDS 194IB P","TDS @20% u/s 194IB","TDS @20% u/s 194IB","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ib"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194IB",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194IB",""
"tds_10_us_194ic","10% TDS 194IC P","TDS @10% u/s 194IC","TDS @10% u/s 194IC","none","purchase","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ic"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194IC",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194IC",""
"tds_20_us_194ic","20% TDS 194IC P","TDS @20% u/s 194IC","TDS @20% u/s 194IC","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194ic"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194IC",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194IC",""
"tds_2_us_194j","2% TDS 194J P","TDS @2% u/s 194J","TDS @2% u/s 194J","none","purchase","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194j"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194J",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194J",""
"tds_10_us_194j","10% TDS 194J P","TDS @10% u/s 194J","TDS @10% u/s 194J","none","purchase","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194j"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194J",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194J",""
"tds_20_us_194j","20% TDS 194J P","TDS @20% u/s 194J","TDS @20% u/s 194J","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194j"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194J",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194J",""
"tds_10_us_194k","10% TDS 194K P","TDS @10% u/s 194K","TDS @10% u/s 194K","none","purchase","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194k"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194K",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194K",""
"tds_20_us_194k","20% TDS 194K P","TDS @20% u/s 194K","TDS @20% u/s 194K","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194k"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194K",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194K",""
"tds_10_us_194la","10% TDS 194LA P","TDS @10% u/s 194LA","TDS @10% u/s 194LA","none","purchase","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194la"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194LA",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194LA",""
"tds_20_us_194la","20% TDS 194LA P","TDS @20% u/s 194LA","TDS @20% u/s 194LA","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194la"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194LA",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194LA",""
"tds_10_us_194lba","10% TDS 194LBA(1) P","TDS @10% u/s 194LBA(1)","TDS @10% u/s 194LBA(1)","none","purchase","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lba1"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194LBA(1)",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194LBA(1)",""
"tds_20_us_194lba","20% TDS 194LBA(1) P","TDS @20% u/s 194LBA(1)","TDS @20% u/s 194LBA(1)","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lba1"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194LBA(1)",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194LBA(1)",""
"tds_10_us_194lbb","10% TDS 194LBB P","TDS @10% u/s 194LBB","TDS @10% u/s 194LBB","none","purchase","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lbb"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194LBB",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194LBB",""
"tds_20_us_194lbb","20% TDS 194LBB P","TDS @20% u/s 194LBB","TDS @20% u/s 194LBB","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lbb"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194LBB",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194LBB",""
"tds_5_us_194lb","5% TDS 194LB P","TDS @5% u/s 194LB","TDS @5% u/s 194LB","none","purchase","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lb"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194LB",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194LB",""
"tds_20_us_194lb","20% TDS 194LB P","TDS @20% u/s 194LB","TDS @20% u/s 194LB","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lb"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194LB",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194LB",""
"tds_10_us_194lbc","10% TDS 194LBC P","TDS @10% u/s 194LBC","TDS @10% u/s 194LBC","none","purchase","percent","-10.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lbc"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194LBC",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194LBC",""
"tds_25_us_194lbc","25% TDS 194LBC P","TDS @25% u/s 194LBC","TDS @25% u/s 194LBC","none","purchase","percent","-25.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lbc"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194LBC",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194LBC",""
"tds_30_us_194lbc","30% TDS 194LBC P","TDS @30% u/s 194LBC","TDS @30% u/s 194LBC","none","purchase","percent","-30.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194lbc"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194LBC",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194LBC",""
"tds_2_us_194m","2% TDS 194M P","TDS @2% u/s 194M","TDS @2% u/s 194M","none","purchase","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194m"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194M",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194M",""
"tds_5_us_194m","5% TDS 194M P","TDS @5% u/s 194M","TDS @5% u/s 194M","none","purchase","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194m"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194M",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194M",""
"tds_20_us_194m","20% TDS 194M P","TDS @20% u/s 194M","TDS @20% u/s 194M","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194m"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194M",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194M",""
"tds_2_us_194n","2% TDS 194N P","TDS @2% u/s 194N","TDS @2% u/s 194N","none","purchase","percent","-2.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194n"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194N",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194N",""
"tds_5_us_194n","5% TDS 194N P","TDS @5% u/s 194N","TDS @5% u/s 194N","none","purchase","percent","-5.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194n"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194N",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194N",""
"tds_20_us_194n","20% TDS 194N P","TDS @20% u/s 194N","TDS @20% u/s 194N","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194n"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194N",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194N",""
"tds_01_us_194o","0.1% TDS 194O P","TDS @0.1% u/s 194O","TDS @0.1% u/s 194O","none","purchase","percent","-0.1","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194o"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194O",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194O",""
"tds_1_us_194o","1% TDS 194O P","TDS @1% u/s 194O","TDS @1% u/s 194O","none","purchase","percent","-1.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194o"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194O",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194O",""
"tds_20_us_194o","20% TDS 194O P","TDS @20% u/s 194O","TDS @20% u/s 194O","none","purchase","percent","-20.0","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194o"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194O",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194O",""
"tds_01_us_194q","0.1% TDS 194Q P","TDS @0.1% u/s 194Q","TDS @0.1% u/s 194Q","none","purchase","percent","-0.1","consu","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_194q"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-194Q",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+194Q",""
"tds_10_4_us_195","10.4% TDS 195 P","TDS @10.4% u/s 195","TDS @10.4% u/s 195","none","purchase","percent","-10.4","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_195"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-195",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+195",""
"tds_15_6_us_195","15.6% TDS 195 P","TDS @15.6% u/s 195","TDS @15.6% u/s 195","none","purchase","percent","-15.6","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_195"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-195",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+195",""
"tds_20_8_us_195","20.8% TDS 195 P","TDS @20.8% u/s 195","TDS @20.8% u/s 195","none","purchase","percent","-20.8","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_195"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-195",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+195",""
"tds_31_2_us_195","31.2% TDS 195 P","TDS @31.2% u/s 195","TDS @31.2% u/s 195","none","purchase","percent","-31.2","service","","tds_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tds_section_195"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11244","-195",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11244","+195",""
"tcs_1_us_206c_1_alfhc","1% TCS 206C(1): A","TCS @1% u/s 206C(1): Alcoholic Liquor for human consumption","1% TCS 206C(1)","sale","","percent","1.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1_alc"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Alcoholic Liquor",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Alcoholic Liquor",""
"tcs_5_us_206c_1_alfhc","5% TCS 206C(1): A","TCS @5% u/s 206C(1): Alcoholic Liquor for human consumption","5% TCS 206C(1)","sale","","percent","5.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1_alc"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Alcoholic Liquor",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Alcoholic Liquor",""
"tcs_5_us_206c_1_tl","5% TCS 206C(1): T L","TCS @5% u/s 206C(1): Tendu leaves","5% TCS 206C(1)","sale","","percent","5.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1_tl"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Tendu leaves",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Tendu leaves",""
"tcs_2_5_us_206c_1_touafl","2.5% TCS 206C(1): Tim","TCS @2.5% u/s 206C(1): Timber obtained under a forest lease","2.5% TCS 206C(1)","sale","","percent","2.5","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1_tim"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Timber (forest lease)",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Timber (forest lease)",""
"tcs_5_us_206c_1_touafl","5% TCS 206C(1): Tim","TCS @5% u/s 206C(1): Timber obtained under a forest lease","5% TCS 206C(1)","sale","","percent","5.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1_tim"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Timber (forest lease)",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Timber (forest lease)",""
"tcs_2_5_us_206c_1_tobamotuafl","2.5% TCS 206C(1): Tim O","TCS @2.5% u/s 206C(1): Timber obtained by any mode other than under a forest lease","2.5% TCS 206C(1)","sale","","percent","2.5","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1_tim_o"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Timber (other than under a forest lease)",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Timber (other than under a forest lease)",""
"tcs_5_us_206c_1_tobamotuafl","5% TCS 206C(1): Tim O","TCS @5% u/s 206C(1): Timber obtained by any mode other than under a forest lease","5% TCS 206C(1)","sale","","percent","5.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1_tim_o"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Timber (other than under a forest lease)",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Timber (other than under a forest lease)",""
"tcs_2_5_us_206c_1_aofpnbtotl","2.5% TCS 206C(1): F O","TCS @2.5% u/s 206C(1): Any other forest produce not being timber or tendu leaves","2.5% TCS 206C(1)","sale","","percent","2.5","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1_fo"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) other forest produce",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) other forest produce",""
"tcs_5_us_206c_1_aofpnbtotl","5% TCS 206C(1): F O","TCS @5% u/s 206C(1): Any other forest produce not being timber or tendu leaves","5% TCS 206C(1)","sale","","percent","5.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1_fo"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) other forest produce",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) other forest produce",""
"tcs_1_us_206c_1_s","1% TCS 206C(1): Sc","TCS @1% u/s 206C(1): Scrap","1% TCS 206C(1)","sale","","percent","1.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1_sc"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Scrap",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Scrap",""
"tcs_5_us_206c_1_s","5% TCS 206C(1): Sc","TCS @5% u/s 206C(1): Scrap","5% TCS 206C(1)","sale","","percent","5.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1_sc"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Scrap",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Scrap",""
"tcs_1_us_206c_1_mbcoloio","1% TCS 206C(1): Min","TCS @1% u/s 206C(1): Minrals, being coal or lignite or iron ore","1% TCS 206C(1)","sale","","percent","1.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1_min"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Minrals",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Minrals",""
"tcs_5_us_206c_1_mbcoloio","5% TCS 206C(1): Min","TCS @5% u/s 206C(1): Minrals, being coal or lignite or iron ore","5% TCS 206C(1)","sale","","percent","5.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1_min"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1) Minrals",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1) Minrals",""
"tcs_2_us_206c_1c_pl","2% TCS 206C(1C): P","TCS @2% u/s 206C(1C): Parking lot","2% TCS 206C(1C)","sale","","percent","2.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1c_p"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1C) Parking lot",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1C) Parking lot",""
"tcs_5_us_206c_1c_pl","5% TCS 206C(1C): P","TCS @5% u/s 206C(1C): Parking lot","5% TCS 206C(1C)","sale","","percent","5.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1c_p"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1C) Parking lot",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1C) Parking lot",""
"tcs_2_us_206c_1c_tp","2% TCS 206C(1C): T","TCS @2% u/s 206C(1C): Toll plaza","2% TCS 206C(1C)","sale","","percent","2.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1c_t"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1C) Toll plaza",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1C) Toll plaza",""
"tcs_5_us_206c_1c_tp","5% TCS 206C(1C): T","TCS @5% u/s 206C(1C): Toll plaza","5% TCS 206C(1C)","sale","","percent","5.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1c_t"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1C) Toll plaza",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1C) Toll plaza",""
"tcs_2_us_206c_1c_maq","2% TCS 206C(1C): M Q","TCS @2% u/s 206C(1C): Mining and quarrying","2% TCS 206C(1C)","sale","","percent","2.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1c_mq"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1C) Mining and quarrying",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1C) Mining and quarrying",""
"tcs_5_us_206c_1c_maq","5% TCS 206C(1C): M Q","TCS @5% u/s 206C(1C): Mining and quarrying","5% TCS 206C(1C)","sale","","percent","5.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1c_mq"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1C) Mining and quarrying",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1C) Mining and quarrying",""
"tcs_1_us_206c_1f_mv","1% TCS 206C(1F): M V","TCS @1% u/s 206C(1F): Motor Vehicle","1% TCS 206C(1F)","sale","","percent","1.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1f_mv"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1F)",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1F)",""
"tcs_5_us_206c_1f_mv","5% TCS 206C(1F): M V","TCS @5% u/s 206C(1F): Motor Vehicle","5% TCS 206C(1F)","sale","","percent","5.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1f_mv"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1F)",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1F)",""
"tcs_5_us_206c_1g_som","5% TCS 206C(1G): R","TCS @5% u/s 206C(1G): Sum of money (above 7 lakhs) for remittance out of India","5% TCS 206C(1G)","sale","","percent","5.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1g_r"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1G) remittance out of India",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1G) remittance out of India",""
"tcs_5_us_206c_1g_soaotpp","5% TCS 206C(1G): O T","TCS @5% u/s 206C(1G): Seller of an overseas tour program package","5% TCS 206C(1G)","sale","","percent","5.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section_206c1g_ot"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1G) overseas tour program",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1G) overseas tour program",""
"tcs_0_1_us_206c_1h_sog","0.1% TCS 206C(1H): G","TCS @0.1% u/s 206C(1H): Sale of Goods","0.1% TCS 206C(1H)","sale","","percent","0.1","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section206c1h_g"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1H)",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1H)",""
"tcs_1_us_206c_1h_sog","1% TCS 206C(1H): G","TCS @1% u/s 206C(1H): Sale of Goods","1% TCS 206C(1H)","sale","","percent","1.0","consu","","tcs_group","","","","","","","100","base","invoice","","","l10n_in_withholding.tcs_section206c1h_g"
"","","","","","","","","","","","","","","","","","100","tax","invoice","p11245","+206C(1H)",""
"","","","","","","","","","","","","","","","","","100","base","refund","","",""
"","","","","","","","","","","","","","","","","","100","tax","refund","p11245","-206C(1H)",""

```

## File: models\account_account.py

```python
from odoo import fields, models


class AccountAccount(models.Model):
    _inherit = 'account.account'

    l10n_in_tds_tcs_section_id = fields.Many2one('l10n_in.section.alert', string="TCS/TDS Section")

```

## File: models\account_chart_template.py

```python
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('in', 'account.account')
    def _get_in_withholding_account_account(self):
        return self._parse_csv('in', 'account.account', module='l10n_in_withholding')

    @template('in', 'account.tax')
    def _get_in_withholding_account_tax(self):
        tax_data = self._parse_csv('in', 'account.tax', module='l10n_in_withholding')
        self._deref_account_tags('in', tax_data)
        return tax_data

    @template('in', 'res.company')
    def _get_in_base_res_company(self):
        return {
            self.env.company.id: {
                'l10n_in_withholding_account_id': 'p100595',
            },
        }

```

## File: models\account_move.py

```python
from odoo import api, models, fields, _, Command
from odoo.tools import SQL
from odoo.tools.date_utils import get_month


class AccountMove(models.Model):
    _inherit = "account.move"

    l10n_in_is_withholding = fields.Boolean(
        string="Is Indian TDS Entry",
        copy=False,
        help="Technical field to identify Indian withholding entry"
    )
    l10n_in_withholding_ref_move_id = fields.Many2one(
        comodel_name='account.move',
        string="Indian TDS Ref Move",
        readonly=True,
        index='btree_not_null',
        copy=False,
        help="Reference move for withholding entry",
    )
    l10n_in_withhold_move_ids = fields.One2many(
        'account.move', 'l10n_in_withholding_ref_move_id',
        string="Indian TDS Entries"
    )
    l10n_in_withholding_line_ids = fields.One2many(
        'account.move.line', 'move_id',
        string="Indian TDS Lines",
        compute='_compute_l10n_in_withholding_line_ids',
    )
    l10n_in_total_withholding_amount = fields.Monetary(
        string="Total Indian TDS Amount",
        compute='_compute_l10n_in_total_withholding_amount',
        help="Total withholding amount for the move",
    )
    l10n_in_tcs_tds_warning = fields.Char('TDC/TCS Warning', compute="_compute_l10n_in_tcs_tds_warning")
    l10n_in_display_higher_tcs_button = fields.Boolean(string="Display higher TCS button", compute="_compute_l10n_in_display_higher_tcs_button")

    # === Compute Methods ===
    @api.depends('line_ids', 'l10n_in_is_withholding')
    def _compute_l10n_in_withholding_line_ids(self):
        # Compute the withholding lines for the move
        for move in self:
            if move.l10n_in_is_withholding:
                move.l10n_in_withholding_line_ids = move.line_ids.filtered('tax_ids')
            else:
                move.l10n_in_withholding_line_ids = False

    def _compute_l10n_in_total_withholding_amount(self):
        for move in self:
            move.l10n_in_total_withholding_amount = sum(move.l10n_in_withhold_move_ids.filtered(
                lambda m: m.state == 'posted').l10n_in_withholding_line_ids.mapped('l10n_in_withhold_tax_amount'))

    def _get_l10n_in_invalid_tax_lines(self):
        self.ensure_one()
        if self.country_code == 'IN' and not self.commercial_partner_id.l10n_in_pan:
            lines = self.env['account.move.line']
            for line in self.invoice_line_ids:
                for tax in line.tax_ids:
                    if (
                        tax.l10n_in_section_id.tax_source_type == 'tcs'
                        and tax.amount != max(tax.l10n_in_section_id.l10n_in_section_tax_ids, key=lambda t: abs(t.amount)).amount
                    ):
                        lines |= line._origin
            return lines

    @api.depends('invoice_line_ids.tax_ids', 'commercial_partner_id.l10n_in_pan')
    def _compute_l10n_in_warning(self):
        super()._compute_l10n_in_warning()
        for move in self:
            warnings = move.l10n_in_warning or {}
            lines = move._get_l10n_in_invalid_tax_lines()
            if lines:
                warnings['lower_tcs_tax'] = {
                        'message': _("As the Partner's PAN missing/invalid apply TCS at the higher rate."),
                        'action_text': _("View Journal Items(s)"),
                        'action': lines._get_records_action(
                            name=_("Journal Items(s)"),
                            target='current',
                            views=[(self.env.ref("l10n_in_withholding.view_move_line_tree_l10n_in").id, "list")],
                            domain=[('id', 'in', lines.ids)]
                        )
                }
                move.l10n_in_warning = warnings

    def action_l10n_in_apply_higher_tax(self):
        self.ensure_one()
        invalid_lines = self._get_l10n_in_invalid_tax_lines()
        for line in invalid_lines:
            updated_tax_ids = []
            for tax in line.tax_ids:
                if tax.l10n_in_section_id.tax_source_type == 'tcs':
                    max_tax = max(
                        tax.l10n_in_section_id.l10n_in_section_tax_ids,
                        key=lambda t: t.amount
                    )
                    updated_tax_ids.append(max_tax.id)
                else:
                    updated_tax_ids.append(tax.id)
            if set(line.tax_ids.ids) != set(updated_tax_ids):
                line.write({'tax_ids': [Command.clear()] + [Command.set(updated_tax_ids)]})

    @api.depends('l10n_in_warning')
    def _compute_l10n_in_display_higher_tcs_button(self):
        for move in self:
            move.l10n_in_display_higher_tcs_button = (
                move.l10n_in_warning
                and move.l10n_in_warning.get('lower_tcs_tax')
            )

    def action_l10n_in_withholding_entries(self):
        self.ensure_one()
        return {
            'name': "TDS Entries",
            'type': 'ir.actions.act_window',
            'res_model': 'account.move',
            'view_mode': 'list,form',
            'domain': [('id', 'in', self.l10n_in_withhold_move_ids.ids)],
        }

    def _get_sections_aggregate_sum_by_pan(self, section_alert, commercial_partner_id):
        self.ensure_one()
        month_start_date, month_end_date = get_month(self.date)
        company_fiscalyear_dates = self.company_id.compute_fiscalyear_dates(self.date)
        fiscalyear_start_date, fiscalyear_end_date = company_fiscalyear_dates['date_from'], company_fiscalyear_dates['date_to']
        default_domain = [
            ('account_id.l10n_in_tds_tcs_section_id', '=', section_alert.id),
            ('move_id.move_type', '!=', 'entry'),
            ('company_id', 'child_of', self.company_id.root_id.id),
            ('parent_state', '=', 'posted')
        ]
        if commercial_partner_id.l10n_in_pan:
            default_domain += [('move_id.commercial_partner_id.l10n_in_pan', '=', commercial_partner_id.l10n_in_pan)]
        else:
            default_domain += [('move_id.commercial_partner_id', '=', commercial_partner_id.id)]
        frequency_domains = {
            'monthly': [('date', '>=', month_start_date), ('date', '<=', month_end_date)],
            'fiscal_yearly': [('date', '>=', fiscalyear_start_date), ('date', '<=', fiscalyear_end_date)],
        }
        aggregate_result = {}
        for frequency, frequency_domain in frequency_domains.items():
            query = self.env['account.move.line']._where_calc(default_domain + frequency_domain)
            result = self.env.execute_query_dict(SQL(
                """
                SELECT COALESCE(sum(account_move_line.balance), 0) as balance,
                       COALESCE(sum(account_move_line.price_total * am.invoice_currency_rate), 0) as price_total
                  FROM %s
                  JOIN account_move AS am ON am.id = account_move_line.move_id
                 WHERE %s
                """,
                query.from_clause,
                query.where_clause)
            )
            aggregate_result[frequency] = result[0]
        return aggregate_result

    def _l10n_in_is_warning_applicable(self, section_id):
        self.ensure_one()
        match section_id.tax_source_type:
            case 'tcs':
                return self.journal_id.type == 'sale'
            case 'tds':
                return (
                    self.journal_id.type == 'purchase'
                    and section_id not in self.l10n_in_withhold_move_ids.filtered(lambda m:
                        m.state == 'posted'
                    ).mapped('line_ids.tax_ids.l10n_in_section_id')
                )
            case _:
                return False

    @api.depends('invoice_line_ids.price_total')
    def _compute_l10n_in_tcs_tds_warning(self):
        def _group_by_section_alert(invoice_lines):
            group_by_lines = {}
            for line in invoice_lines:
                group_key = line.account_id.l10n_in_tds_tcs_section_id
                if group_key and not line.company_currency_id.is_zero(line.price_total):
                    group_by_lines.setdefault(group_key, [])
                    group_by_lines[group_key].append(line)
            return group_by_lines

        def _is_section_applicable(section_alert, threshold_sums, invoice_currency_rate, lines):
            lines_total = sum(
                    (line.price_total * invoice_currency_rate) if section_alert.consider_amount == 'total_amount' else line.balance
                    for line in lines
                )
            if section_alert.is_aggregate_limit:
                aggregate_period_key = section_alert.consider_amount == 'total_amount' and 'price_total' or 'balance'
                aggregate_total = threshold_sums.get(section_alert.aggregate_period, {}).get(aggregate_period_key)
                if move.state == 'draft':
                    aggregate_total += lines_total
                if aggregate_total > section_alert.aggregate_limit:
                    return True
            return (
                section_alert.is_per_transaction_limit
                and lines_total > section_alert.per_transaction_limit
            )

        for move in self:
            if move.country_code == 'IN' and move.move_type in ['in_invoice', 'out_invoice']:
                warning = set()
                commercial_partner_id = move.commercial_partner_id
                existing_section = (self.l10n_in_withhold_move_ids.line_ids + move.line_ids).tax_ids.l10n_in_section_id
                for section_alert, lines in _group_by_section_alert(move.invoice_line_ids).items():
                    if (
                        (section_alert not in existing_section
                        or [line for line in lines if section_alert not in line.tax_ids.l10n_in_section_id])
                        and move._l10n_in_is_warning_applicable(section_alert)
                        and _is_section_applicable(
                            section_alert,
                            move._get_sections_aggregate_sum_by_pan(
                                section_alert,
                                commercial_partner_id
                            ),
                            move.invoice_currency_rate,
                            lines
                        )
                    ):
                        warning.add(section_alert.id)
                warning_sections = self.env['l10n_in.section.alert'].browse(warning)
                if warning_sections:
                    move.l10n_in_tcs_tds_warning = warning_sections._get_warning_message()
                else:
                    move.l10n_in_tcs_tds_warning = False
            else:
                move.l10n_in_tcs_tds_warning = False

```

## File: models\account_move_line.py

```python
from odoo import fields, models, api


class AccountMoveLine(models.Model):
    _inherit = "account.move.line"

    l10n_in_withhold_tax_amount = fields.Monetary(string="TDS Tax Amount", compute='_compute_withhold_tax_amount')

    @api.depends('tax_ids')
    def _compute_withhold_tax_amount(self):
        # Compute the withhold tax amount for the withholding lines
        withholding_lines = self.filtered('move_id.l10n_in_is_withholding')
        (self - withholding_lines).l10n_in_withhold_tax_amount = False
        for line in withholding_lines:
            line.l10n_in_withhold_tax_amount = line.currency_id.round(abs(line.price_total - line.price_subtotal))

```

## File: models\account_payment.py

```python
from odoo import models, fields


class AccountPayment(models.Model):
    _inherit = "account.payment"

    l10n_in_total_withholding_amount = fields.Monetary(related='move_id.l10n_in_total_withholding_amount')
    l10n_in_withhold_move_ids = fields.One2many(related='move_id.l10n_in_withhold_move_ids')

    def action_l10n_in_withholding_entries(self):
        self.ensure_one()
        return {
            'name': "TDS Entries",
            'type': 'ir.actions.act_window',
            'res_model': 'account.move',
            'view_mode': 'list,form',
            'domain': [('id', 'in', self.l10n_in_withhold_move_ids.ids)],
        }

```

## File: models\account_tax.py

```python
from odoo import fields, models


class AccountTax(models.Model):
    _inherit = 'account.tax'

    l10n_in_tds_tax_type = fields.Selection([
        ('sale', 'Sale'),
        ('purchase', 'Purchase')
    ], string="TDS Tax Type")
    l10n_in_section_id = fields.Many2one('l10n_in.section.alert', string="Section")

```

## File: models\l10n_in_section_alert.py

```python
from odoo import api, fields, models, _


class L10nInSectionAlert(models.Model):
    _name = "l10n_in.section.alert"
    _description = "indian section alert"

    name = fields.Char("Section Name")
    tax_source_type = fields.Selection([
            ('tds', 'TDS'),
            ('tcs', 'TCS'),
        ], string="Tax Source Type")
    consider_amount = fields.Selection([
            ('untaxed_amount', 'Untaxed Amount'),
            ('total_amount', 'Total Amount'),
        ], string="Consider", default='untaxed_amount', required=True)
    is_per_transaction_limit = fields.Boolean("Per Transaction")
    per_transaction_limit = fields.Float("Per Transaction limit")
    is_aggregate_limit = fields.Boolean("Aggregate")
    aggregate_limit = fields.Float("Aggregate limit")
    aggregate_period = fields.Selection([
            ('monthly', 'Monthly'),
            ('fiscal_yearly', 'Financial Yearly'),
        ], string="Aggregate Period", default='fiscal_yearly')
    l10n_in_section_tax_ids = fields.One2many("account.tax", "l10n_in_section_id", string="Taxes")

    _sql_constraints = [
        ('per_transaction_limit', 'CHECK(per_transaction_limit >= 0)', 'Per transaction limit must be positive'),
        ('aggregate_limit', 'CHECK(aggregate_limit >= 0)', 'Aggregate limit must be positive'),
    ]

    @api.depends('tax_source_type')
    def _compute_display_name(self):
        for record in self:
            record.display_name = f"{record.tax_source_type.upper()} {record.name or ''}" if record.tax_source_type else f"{record.name or ''}"

    def _get_warning_message(self):
        warning = ", ".join(self.mapped('name'))
        section_type = next(iter(set(self.mapped('tax_source_type')))).upper()
        action = 'collect' if section_type == 'TCS' else 'deduct'
        return _("It's advisable to %(action)s %(section_type)s u/s %(warning)s on this transaction.",
            action=action,
            section_type=section_type,
            warning=warning
        )

```

## File: models\res_company.py

```python
from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_in_withholding_account_id = fields.Many2one(
        comodel_name='account.account',
        string="TDS Account",
        check_company=True,
    )
    l10n_in_withholding_journal_id = fields.Many2one(
        comodel_name='account.journal',
        string="TDS Journal",
        check_company=True,
    )

```

## File: models\res_config_settings.py

```python
from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_in_withholding_account_id = fields.Many2one(
        related='company_id.l10n_in_withholding_account_id',
        readonly=False,
    )
    l10n_in_withholding_journal_id = fields.Many2one(
        related='company_id.l10n_in_withholding_journal_id',
        readonly=False,
    )

```

## File: models\__init__.py

```python
from . import account_chart_template
from . import account_move
from . import account_move_line
from . import account_payment
from . import account_tax
from . import res_company
from . import res_config_settings
from . import account_account
from . import l10n_in_section_alert

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
l10n_in_withholding.access_l10n_in_withhold_wizard,access_l10n_in_withhold_wizard,l10n_in_withholding.model_l10n_in_withhold_wizard,account.group_account_invoice,1,1,1,1
l10n_in_withholding.access_l10n_in_withhold_wizard_line,access_l10n_in_withhold_wizard_line,l10n_in_withholding.model_l10n_in_withhold_wizard_line,account.group_account_invoice,1,1,1,1
access_l10n_in_section_alert_account_readonly,l10n_in.section.alert.account.readonly,model_l10n_in_section_alert,account.group_account_readonly,1,0,0,0
access_l10n_in_section_alert_account_manager,l10n_in.section.alert.account.manager,model_l10n_in_section_alert,account.group_account_manager,1,1,1,1

```

## File: views\account_account_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_account_tds_tcs_view_form_inherit" model="ir.ui.view">
        <field name="name">account.account.tds.tcs.view.form.inherit</field>
        <field name="model">account.account</field>
        <field name="inherit_id" ref="account.view_account_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='tax_ids']" position="after">
                <field name="l10n_in_tds_tcs_section_id" invisible="company_fiscal_country_code != 'IN'"/>
            </xpath>
        </field>
    </record>

    <record id="account_account_tds_tcs_view_tree_inherit" model="ir.ui.view">
        <field name="name">account.account.tds.tcs.view.list.inherit</field>
        <field name="model">account.account</field>
        <field name="inherit_id" ref="account.view_account_list"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='account_type']" position="after">
                <field name="l10n_in_tds_tcs_section_id" optional="hide"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\account_move_line_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_move_line_tree_l10n_in" model="ir.ui.view">
        <field name="name">account.move.line.tree.l10n.in</field>
        <field name="model">account.move.line</field>
        <field name="inherit_id" ref="l10n_in.view_move_line_tree_hsn_l10n_in"/>
        <field name="mode">primary</field>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='l10n_in_hsn_code']" position="after">
                <field name="tax_ids" widget="many2many_tags" domain="[('type_tax_use', '=', 'sale')]"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\account_move_views.xml

```xml
<odoo>
    <record id="account_move_view_form_inherit_l10n_in_withholding" model="ir.ui.view">
        <field name="name">account.move.form.inherit.l10n_in_withholding</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <button name="%(l10n_in_withholding_entry_form_action)d" string="TDS Entry" type="action" class="btn btn-secondary float-end"
                        invisible="country_code != 'IN' or move_type not in ('out_invoice', 'in_invoice', 'out_refund', 'in_refund') or state != 'posted'"/>
                <button name="action_l10n_in_apply_higher_tax" string="Apply Higher TCS" type="object" class="btn btn-secondary float-end"
                        invisible="not l10n_in_display_higher_tcs_button"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="action_l10n_in_withholding_entries"
                        class="oe_stat_button"
                        type="object"
                        icon="fa-list-alt"
                        invisible="not l10n_in_withhold_move_ids">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">TDS</span>
                        <span class="o_stat_value"><field name="l10n_in_total_withholding_amount"/></span>
                    </div>
                </button>
            </xpath>
            <xpath expr="//notebook/page[@id='aml_tab']" position="before">
                <page name="withholding_tab" string="TDS Information" invisible="not l10n_in_withholding_line_ids">
                    <field name="l10n_in_withholding_line_ids" nolabel="1" colspan="4">
                        <list editable="bottom" string="TDS Information">
                            <field name="tax_ids" string="Tax" widget="many2many_tags"/>
                            <field name="price_subtotal" string="Base Amount" sum="Total"/>
                            <field name="l10n_in_withhold_tax_amount" string="TDS Amount" sum="Total"/>
                        </list>
                    </field>
                </page>
            </xpath>
            <xpath expr="//sheet" position="before">
                <div class="alert alert-warning" role="alert" invisible="not l10n_in_tcs_tds_warning">
                    <field name="l10n_in_tcs_tds_warning" readonly="1"/>
                </div>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\account_payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_account_payment_form_inherit_l10n_in_withholding" model="ir.ui.view">
        <field name="name">account.payment.form.inherit.l10n_in_withholding</field>
        <field name="model">account.payment</field>
        <field name="inherit_id" ref="account.view_account_payment_form"/>
        <field name="arch" type="xml">
            <xpath expr="//header" position="inside">
                <button name="%(l10n_in_withholding_entry_form_action)d" string="TDS Entry" type="action" class="btn btn-secondary float-end"
                        invisible="country_code != 'IN' or state not in ('in_process', 'paid') or is_reconciled"/>
            </xpath>
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="action_l10n_in_withholding_entries"
                        class="oe_stat_button"
                        type="object"
                        icon="fa-list-alt"
                        invisible="not l10n_in_withhold_move_ids">
                    <div class="o_field_widget o_stat_info">
                        <span class="o_stat_text">TDS</span>
                        <span class="o_stat_value"><field name="l10n_in_total_withholding_amount"/></span>
                    </div>
                </button>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\account_tax_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_tax_form_inherited_l10n_in_withholding" model="ir.ui.view">
        <field name="name">account.tax.form.inherited.l10n_in_withholding</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='country_id']" position="after">
                <field name="l10n_in_tds_tax_type" invisible="country_code != 'IN'"/>
            </xpath>
            <xpath expr="//field[@name='tax_group_id']" position="after">
                <field name="l10n_in_section_id" invisible="country_code != 'IN'"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\l10n_in_section_alert_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="l10n_in_section_alert_view_tree" model="ir.ui.view">
        <field name="name">l10n_in.section.alert.view.list</field>
        <field name="model">l10n_in.section.alert</field>
        <field name="arch" type="xml">
            <list string="Section">
                <field name="name"/>
                <field name="tax_source_type"/>
                <field name="consider_amount"/>
                <field name="per_transaction_limit"/>
                <field name="aggregate_limit"/>
            </list>
        </field>
    </record>

    <record id="l10n_in_section_alert_view_form" model="ir.ui.view">
        <field name="name">l10n_in.section.alert.view.form</field>
        <field name="model">l10n_in.section.alert</field>
        <field name="arch" type="xml">
            <form string="Section">
                <sheet>
                    <div class="oe_title">
                        <label for="name" string="Section Name"/>
                        <h1>
                            <field name="name"/>
                        </h1>
                    </div>
                    <group class="w-50" string="Threshold limits">
                        <field name="consider_amount"/>
                        <label for="is_per_transaction_limit"/>
                        <div>
                            <field class="w-25" name="is_per_transaction_limit" widget="boolean_toggle"/>
                            <field class="w-25 text-center oe_inline" name="per_transaction_limit" invisible="not is_per_transaction_limit"/>
                        </div>
                        <label for="is_aggregate_limit"/>
                        <div>
                            <field class="w-25" name="is_aggregate_limit" widget="boolean_toggle"/>
                            <field class="w-25 text-center oe_inline" name="aggregate_limit" invisible="not is_aggregate_limit"/>
                            <field class="w-25" name="aggregate_period" invisible="not is_aggregate_limit"/>
                        </div>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="l10n_in_section_alert_action" model="ir.actions.act_window">
        <field name="name">Section</field>
        <field name="res_model">l10n_in.section.alert</field>
        <field name="view_mode">list,form</field>
    </record>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<odoo>
    <record id="res_config_settings_view_form_inherit_l10n_in_withholding" model="ir.ui.view">
        <field name="name">res.config.settings.form.inherit.l10n_in_withholding</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <block id="default_accounts" position="inside">
                <setting string="India TDS Control:">
                    <div class="content-group">
                        <div class="row mt8">
                            <label for="l10n_in_withholding_journal_id" class="col-lg-5 o_light_label" string="Journal"/>
                            <field name="l10n_in_withholding_journal_id" domain="[('type', '=', 'general')]"/>
                        </div>
                        <div class="row mt8">
                            <label for="l10n_in_withholding_account_id" class="col-lg-5 o_light_label" string="Account"/>
                            <field name="l10n_in_withholding_account_id"/>
                        </div>
                    </div>
                </setting>
            </block>
        </field>
    </record>
</odoo>

```

## File: wizard\l10n_in_withhold_wizard.py

```python
from markupsafe import Markup

from odoo import _, api, Command, fields, models
from odoo.exceptions import ValidationError, UserError
from odoo.tools import float_compare


class L10nInWithholdWizard(models.TransientModel):
    _name = 'l10n_in.withhold.wizard'
    _description = "Withhold Wizard"
    _check_company_auto = True

    @api.model
    def default_get(self, fields_list):
        result = super().default_get(fields_list)
        active_model = self._context.get('active_model')
        active_ids = self._context.get('active_ids', [])
        if len(active_ids) > 1:
            raise UserError(_("You can only create a withhold for only one record at a time."))
        if active_model not in ('account.move', 'account.payment') or not active_ids:
            raise UserError(_("TDS must be created from an Invoice or a Payment."))
        active_record = self.env[active_model].browse(active_ids)
        result['reference'] = _("TDS of %s", active_record.name)
        if active_model == 'account.move':
            if active_record.move_type not in ('out_invoice', 'out_refund', 'in_invoice', 'in_refund') or active_record.state != 'posted':
                raise UserError(_("TDS must be created from Posted Customer Invoices, Customer Credit Notes, Vendor Bills or Vendor Refunds."))
            result['related_move_id'] = active_record.id
        elif active_model == 'account.payment':
            if not active_record.partner_id:
                type_name = _("Vendor Payment") if active_record.partner_type == 'supplier' else _("Customer Payment")
                raise UserError(_("Please set a partner on the %s before creating a withhold.", type_name))
            result['related_payment_id'] = active_record.id
        return result

    reference = fields.Char(string="Reference")
    type_name = fields.Char(string="Type", compute='_compute_type_name')
    related_move_id = fields.Many2one(
        comodel_name='account.move',
        string="Invoice/Bill",
        readonly=True,
    )
    related_payment_id = fields.Many2one(
        comodel_name='account.payment',
        string="Payment",
        readonly=True,
    )
    company_id = fields.Many2one(
        comodel_name='res.company',
        string="Company",
        compute='_compute_company_id'
    )
    currency_id = fields.Many2one(
        related='company_id.currency_id',
        string="Currency",
    )
    journal_id = fields.Many2one(
        comodel_name='account.journal',
        string="Journal",
        compute='_compute_journal', precompute=True,
        readonly=False, store=True,
        required=True,
        check_company=True,
    )
    date = fields.Date(
        string="Date",
        default=fields.Date.context_today,
    )
    l10n_in_tds_tax_type = fields.Char(
        string="Indian Tax Type",
        compute='_compute_l10n_in_tds_tax_type'
    )
    withhold_line_ids = fields.One2many(
        comodel_name='l10n_in.withhold.wizard.line',
        inverse_name='withhold_id',
        string="TDS Lines",
        readonly=False,
        store=True,
    )
    l10n_in_withholding_warning = fields.Json(string="Withholding warning", compute='_compute_l10n_in_withholding_warning')

    #  ===== Computes =====
    @api.depends('related_move_id', 'related_payment_id')
    def _compute_l10n_in_tds_tax_type(self):
        for wizard in self:
            withhold_type = wizard._get_withhold_type()
            l10n_in_tds_tax_type = False
            if withhold_type in ('in_withhold', 'in_refund_withhold'):
                l10n_in_tds_tax_type = 'purchase'
            elif withhold_type in ('out_withhold', 'out_refund_withhold'):
                l10n_in_tds_tax_type = 'sale'
            wizard.l10n_in_tds_tax_type = l10n_in_tds_tax_type

    @api.depends('related_move_id', 'related_payment_id')
    def _compute_type_name(self):
        for wizard in self:
            if wizard.related_payment_id:
                wizard.type_name = _("Vendor Payment") if wizard.related_payment_id.partner_type == 'supplier' else _("Customer Payment")
            else:
                wizard.type_name = wizard.related_move_id.type_name

    @api.depends('related_move_id', 'related_payment_id')
    def _compute_company_id(self):
        for wizard in self:
            wizard.company_id = wizard.related_move_id.company_id or wizard.related_payment_id.company_id

    @api.depends('company_id')
    def _compute_journal(self):
        for wizard in self:
            wizard.journal_id = wizard.company_id.parent_ids.l10n_in_withholding_journal_id[-1:] or \
                                wizard.env['account.journal'].search([*self.env['account.journal']._check_company_domain(wizard.company_id), ('type', '=', 'general')], limit=1)

    @api.depends('related_payment_id', 'related_move_id', 'l10n_in_tds_tax_type', 'withhold_line_ids')
    def _compute_l10n_in_withholding_warning(self):
        for wizard in self:
            warnings = {}
            if wizard.l10n_in_tds_tax_type == 'purchase' and not wizard.related_move_id.commercial_partner_id.l10n_in_pan and any(
                    line.tax_id.amount != max(line.tax_id.l10n_in_section_id.l10n_in_section_tax_ids, key=lambda t: abs(t.amount)).amount
                    for line in wizard.withhold_line_ids
                ):
                warnings['lower_tds_tax'] = {
                    'message': _("As the Partner's PAN missing/invalid, it's advisable to apply TDS at the higher rate.")
                    }
            precision = self.currency_id.decimal_places
            if wizard.related_move_id and float_compare(wizard.related_move_id.amount_untaxed, sum(line.base for line in wizard.withhold_line_ids), precision_digits=precision) < 0:
                message = _("The base amount of TDS lines is greater than the amount of the %s", wizard.type_name)
                warnings['lower_move_amount'] = {
                    'message': message
                }
            elif wizard.related_payment_id and float_compare(wizard.related_payment_id.amount, sum(line.base for line in wizard.withhold_line_ids), precision_digits=precision) < 0:
                message = _("The base amount of TDS lines is greater than the untaxed amount of the %s", wizard.type_name)
                warnings['lower_payment_amount'] = {
                    'message': message
                }
            wizard.l10n_in_withholding_warning = warnings

    def _get_withhold_type(self):
        if self.related_move_id:
            move_type = self.related_move_id.move_type
            withhold_type = {
                'out_invoice': 'out_withhold',
                'in_invoice': 'in_withhold',
                'out_refund': 'out_refund_withhold',
                'in_refund': 'in_refund_withhold',
            }[move_type]
        else:
            withhold_type = 'in_withhold' if self.related_payment_id.partner_type == 'supplier' else 'out_withhold'
        return withhold_type

    # ===== MOVE CREATION METHODS =====
    def action_create_and_post_withhold(self):
        self.ensure_one()
        withholding_account_id = self.company_id.l10n_in_withholding_account_id
        self._validate_withhold_data_on_post(withholding_account_id)

        # Withhold creation and posting
        vals = self._prepare_withhold_header()
        move_lines = self._prepare_withhold_move_lines(withholding_account_id)
        vals['line_ids'] = [Command.create(line) for line in move_lines]
        withhold = self.with_company(self.company_id).env['account.move'].create(vals)
        withhold.action_post()

        # If the withhold is created from a payment, there is no need to reconcile
        if not self.related_payment_id:
            wh_reconc = withhold.line_ids.filtered(
                lambda l: l.account_id.account_type in ('asset_receivable', 'liability_payable'))
            inv_reconc = self.related_move_id.line_ids.filtered(
                lambda l: l.account_id.account_type in ('asset_receivable', 'liability_payable') and not l.reconciled)
            (inv_reconc + wh_reconc).reconcile()
        related_record = self.related_move_id or self.related_payment_id
        withhold.message_post(
            body=Markup("%s %s: <a href='#' data-oe-model='%s' data-oe-id='%s'>%s</a>") % (
                _("TDS created from"),
                self.type_name,
                related_record._name,
                related_record.id,
                related_record.name
            ))
        return withhold

    def _prepare_withhold_header(self):
        """ Prepare the header for the withhold entry """
        vals = {
            'date': self.date,
            'journal_id': self.journal_id.id,
            'partner_id': self.related_move_id.partner_id.id or self.related_payment_id.partner_id.id,
            'move_type': 'entry',
            'ref': self.reference,
            'l10n_in_is_withholding': True,
            'l10n_in_withholding_ref_move_id': self.related_move_id.id or self.related_payment_id.move_id.id,
        }
        return vals

    def _prepare_withhold_move_lines(self, withholding_account_id):
        """
        Prepare the move lines for the withhold entry
        """
        def append_vals(quantity, price_unit, debit, credit, account_id, tax_ids):
            return {
                'quantity': quantity,
                'price_unit': price_unit,
                'debit': debit,
                'credit': credit,
                'account_id': account_id.id,
                'tax_ids': tax_ids,
            }

        vals = []
        total_amount = 0
        total_tax = 0

        partner = self.related_move_id.partner_id or self.related_payment_id.partner_id
        withhold_type = self._get_withhold_type()

        if withhold_type in ('in_withhold', 'in_refund_withhold'):
            partner_account = partner.property_account_payable_id
        else:
            partner_account = partner.property_account_receivable_id

        # Create move lines for each withhold line with the withholding tax and the base amount
        for line in self.withhold_line_ids:
            debit = line.base if withhold_type in ('in_withhold', 'out_refund_withhold') else 0.0
            credit = 0.0 if withhold_type in ('in_withhold', 'out_refund_withhold') else line.base
            vals.append(append_vals(1.0, line.base, debit, credit, withholding_account_id, [Command.set(line.tax_id.ids)]))
            total_amount += line.base
            total_tax += line.amount

        # Create move line for the sum of all withhold lines (total amount)
        debit = 0.0 if withhold_type in ('in_withhold', 'out_refund_withhold') else total_amount
        credit = total_amount if withhold_type in ('in_withhold', 'out_refund_withhold') else 0.0
        vals.append(append_vals(1.0, total_amount, debit, credit, withholding_account_id, False))

        # Create move line for the sum of all withhold taxes (total tax)
        debit = total_tax if withhold_type in ('in_withhold', 'out_refund_withhold') else 0.0
        credit = 0.0 if withhold_type in ('in_withhold', 'out_refund_withhold') else total_tax
        vals.append(append_vals(1.0, total_tax, debit, credit, partner_account, False))

        return vals

    def _validate_withhold_data_on_post(self, withholding_account_id):
        if not withholding_account_id:
            raise UserError(_("Please configure the withholding account from the settings"))
        if not self.withhold_line_ids:
            raise ValidationError(_("You must input at least one withhold line"))


class L10nInWithholdWizardLine(models.TransientModel):
    _name = 'l10n_in.withhold.wizard.line'
    _description = "Withhold Wizard Lines"

    base = fields.Monetary(string="Base")
    currency_id = fields.Many2one(related='withhold_id.currency_id')
    l10n_in_tds_tax_type = fields.Char(related='withhold_id.l10n_in_tds_tax_type')
    withhold_id = fields.Many2one(comodel_name='l10n_in.withhold.wizard', required=True)
    tax_id = fields.Many2one(
        comodel_name='account.tax',
        string="TDS Tax",
        required=True,
    )
    amount = fields.Monetary(
        string="TDS Amount",
        compute='_compute_amount',
        store=True,
    )

    #  ===== Constraints =====
    @api.constrains('base', 'amount')
    def _check_amounts(self):
        for line in self:
            precision = line.currency_id.decimal_places
            if float_compare(line.amount, 0.0, precision_digits=precision) <= 0:
                raise ValidationError(_("Negative or zero values are not allowed in amount for withhold lines"))
            if float_compare(line.base, 0.0, precision_digits=precision) <= 0:
                raise ValidationError(_("Negative or zero values are not allowed in base for withhold lines"))

    @api.depends('tax_id', 'base')
    def _compute_amount(self):
        # Recomputes amount according to "base amount" and tax percentage
        for line in self:
            tax_amount = 0.0
            if line.tax_id:
                tax_amount = line._tax_compute_all_helper(line.base, line.tax_id)
            line.amount = tax_amount

    # === Helper methods ====
    @api.model
    def _tax_compute_all_helper(self, base, tax_id):
        # Computes the withholding tax amount provided a base and a tax
        # It is equivalent to: amount = self.base * self.tax_id.amount / 100
        taxes_res = tax_id.compute_all(
            base,
            currency=tax_id.company_id.currency_id,
            quantity=1.0,
            product=False,
            partner=False,
            is_refund=False,
        )
        tax_amount = taxes_res['total_included'] - taxes_res['total_excluded']
        tax_amount = abs(tax_amount)
        return tax_amount

```

## File: wizard\l10n_in_withhold_wizard.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="l10n_in_withholding_entry_form_action" model="ir.actions.act_window">
        <field name="name">Create TDS Entry</field>
        <field name="res_model">l10n_in.withhold.wizard</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>

    <record id="tds_entry_view_form" model="ir.ui.view">
        <field name="name">l10n_in.withhold.wizard.view.form</field>
        <field name="model">l10n_in.withhold.wizard</field>
        <field name="arch" type="xml">
            <form>
                <div class="alert alert-warning mt-1 mb-1" role="alert" invisible="not l10n_in_withholding_warning">
                    <div>
                        <field name="l10n_in_withholding_warning" widget="actionable_errors"/>
                    </div>
                </div>
                <sheet>
                    <group>
                        <group id="header_left_group">
                            <field name="related_move_id" invisible="1"/> <!-- used to compute the company_id -->
                            <field name="related_payment_id" invisible="1"/> <!-- used to compute the company_id -->
                            <field name="date"/>
                            <field name="journal_id" domain="[('type', '=', 'general')]"/>
                        </group>
                        <group id="header_right_group">
                            <field name="reference"/>
                        </group>
                    </group>
                    <notebook>
                        <page string="TDS Tax Details">
                            <field name="withhold_line_ids">
                                <list editable="bottom">
                                    <field name="currency_id" column_invisible="True"/> <!-- used to display the currency symbol -->
                                    <field name="tax_id" domain="[('l10n_in_tds_tax_type', '=', l10n_in_tds_tax_type)]"/>
                                    <field name="base" sum="Total Base" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                                    <field name="amount" sum="Total Amount" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                                </list>
                            </field>
                        </page>
                    </notebook>
                </sheet>
                <footer>
                    <button string="Confirm" type="object" name="action_create_and_post_withhold" class="btn-primary"/>
                    <button string="Discard" special="cancel" class="btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>

</odoo>

```

## File: wizard\__init__.py

```python
from . import l10n_in_withhold_wizard

```

