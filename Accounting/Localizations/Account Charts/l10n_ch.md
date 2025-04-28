# Odoo Module: l10n_ch

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report
from . import wizard


def init_settings(env):
    '''If the company is localized in Switzerland, activate the cash rounding by default.
    '''
    # The cash rounding is activated by default only if the company is localized in Switzerland or Liechtenstein.
    for company in env['res.company'].search([('partner_id.country_id.code', 'in', ["CH", "LI"])]):
        config_wizard = env['res.config.settings'].create({
            'company_id': company.id,
            'group_cash_rounding': True
        })
        # We need to call execute, otherwise the "implied_group" in fields are not processed.
        config_wizard.execute()
        config_wizard.unlink()

def post_init(env):
    init_settings(env)

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Switzerland - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/switzerland.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['ch'],
    'description': """
Swiss localization
==================
This module defines a chart of account for Switzerland (Swiss PME/KMU 2015), taxes and enables the generation of a QR-bill when you print an invoice or send it by mail.
The QR bill is attached to the invoice and eases its payment.

A QR-bill will be generated if:
    - The partner set on your invoice has a complete address (street, city, postal code and country) in Switzerland
    - The option to generate the Swiss QR-code is selected on the invoice (done by default)
    - A correct account number/QR IBAN is set on your bank journal
    - (when using a QR-IBAN): the payment reference of the invoice is a QR-reference

The generation of the QR-bill is automatic if you meet the previous criteria. The QR-bill will be appended after the invoice when printing or sending by mail. 

    """,
    'version': '11.3',
    'category': 'Accounting/Localizations/Account Charts',
    'depends': [
        'account',
        'base_iban',
        'l10n_din5008',
    ],
    'data': [
        'security/ir.model.access.csv',
        'data/account_tax_report_data.xml',
        'report/swissqr_report.xml',
        'views/res_bank_view.xml',
        'views/account_invoice.xml',
        'views/setup_wizard_views.xml',
        'views/qr_invoice_wizard_view.xml',
        'views/account_payment_view.xml',
    ],
    'demo': [
        'demo/account_cash_rounding.xml',
        'demo/demo_company.xml',
        'demo/res_partner_demo.xml',
    ],
    'post_init_hook': 'post_init',
    'assets': {
        'web.report_assets_common': [
            'l10n_ch/static/src/scss/**/*',
        ],
    }
,
    'license': 'LGPL-3',
}

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ch"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_chiffre_af" model="account.report.line">
                <field name="name">I. TURNOVER</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_chtax_200" model="account.report.line">
                        <field name="name">200 - Total amount of agreed or collected consideration incl. from supplies opted for taxation, transfer of supplies acc. to the notification procedure and supplies provided abroad (worldwide turnover)</field>
                        <field name="code">tax_ch_200</field>
                        <field name="aggregation_formula">tax_ch_302a.balance + tax_ch_303a.balance + tax_ch_312a.balance + tax_ch_313a.balance + tax_ch_342a.balance + tax_ch_343a.balance + tax_ch_205.balance + tax_ch_289.balance</field>
                    </record>
                    <record id="account_tax_report_line_chtax_205" model="account.report.line">
                        <field name="name">205 - Consideration reported in Ref. 200 from supplies exempt from the tax without credit (art. 21) where the option for their taxation according to art. 22 has been exercised</field>
                        <field name="code">tax_ch_205</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_205_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">205</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_220_289" model="account.report.line"> <!-- the xml is as it is for historical reasons but it does represent box 220 only -->
                        <field name="name">220 - Supplies exempt from the tax (e.g. export, art. 23) and supplies provided to institutional and individual beneficiaries that are exempt from liability for tax (art. 107 para. 1 lit. a)</field>
                        <field name="code">tax_ch_220</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_220_289_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">220</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_221" model="account.report.line">
                        <field name="name">221 - Supplies provided abroad</field>
                        <field name="code">tax_ch_221</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_221_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">221</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_225" model="account.report.line">
                        <field name="name">225 - Transfer of supplies according to the notification procedure (art. 38, please submit Form 764)</field>
                        <field name="code">tax_ch_225</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_225_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">225</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_230" model="account.report.line">
                        <field name="name">230 - Supplies provided on Swiss territory exempt from the tax without credit (art. 21) and where the option for their taxation according to art. 22 has not been exercised</field>
                        <field name="code">tax_ch_230</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_230_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">230</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_235" model="account.report.line">
                        <field name="name">235 - Reduction of consideration (discounts, rebates etc.)</field>
                        <field name="code">tax_ch_235</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_235_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">235</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_280" model="account.report.line">
                        <field name="name">280 - Miscellaneous (e.g. land value, purchase prices in case of margin taxation)</field>
                        <field name="code">tax_ch_280</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_280_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">280</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_289" model="account.report.line">
                        <field name="name">289 - Deductions (Total Ref. 220 to 280)</field>
                        <field name="code">tax_ch_289</field>
                        <field name="aggregation_formula">tax_ch_220.balance + tax_ch_221.balance + tax_ch_225.balance + tax_ch_230.balance + tax_ch_235.balance + tax_ch_280.balance</field>
                    </record>
                    <record id="account_tax_report_line_chtax_299" model="account.report.line">
                        <field name="name">299 - Taxable turnover (Ref. 200 minus Ref. 289)</field>
                        <field name="aggregation_formula">tax_ch_200.balance - tax_ch_289.balance</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_calc_impot" model="account.report.line">
                <field name="name">II. TAX CALCULATION</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_supplies_1" model="account.report.line">
                        <field name="name">Supplies CHF from 01.01.2024</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_chtax_303a" model="account.report.line">
                                <field name="name">303a - Standard rate (8,1%): Supplies CHF from 01.01.2024</field>
                                <field name="code">tax_ch_303a</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_303a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">303a</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_chtax_313a" model="account.report.line">
                                <field name="name">313a - Reduced rate (2,6%): Supplies CHF from 01.01.2024</field>
                                <field name="code">tax_ch_313a</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_313a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">313a</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_chtax_343a" model="account.report.line">
                                <field name="name">343a - Accommodation rate (3,8%): Supplies CHF from 01.01.2024</field>
                                <field name="code">tax_ch_343a</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_343a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">343a</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_chtax_383a" model="account.report.line">
                                <field name="name">383a - Acquisition tax: Supplies CHF from 01.01.2024</field>
                                <field name="code">tax_ch_383a</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_383a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">383a</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_supplies_2" model="account.report.line">
                        <field name="name">Supplies CHF to 31.12.2023</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_chtax_302a" model="account.report.line">
                                <field name="name">302a - Standard rate (7,7%): Supplies CHF to 31.12.2023</field>
                                <field name="code">tax_ch_302a</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_302a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">302a</field>
                                    </record>
                                </field>
                            </record>
                             <record id="account_tax_report_line_chtax_312a" model="account.report.line">
                                <field name="name">312a - Reduced rate (2,5%): Supplies CHF to 31.12.2023</field>
                                <field name="code">tax_ch_312a</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_312a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">312a</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_chtax_342a" model="account.report.line">
                                <field name="name">342a - Accommodation rate (3,7%): Supplies CHF to 31.12.2023</field>
                                <field name="code">tax_ch_342a</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_342a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">342a</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_chtax_382a" model="account.report.line">
                                <field name="name">382a - Acquisition tax: Supplies CHF to 31.12.2023</field>
                                <field name="code">tax_ch_382a</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_382a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">382a</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_tax_amount_1" model="account.report.line">
                        <field name="name">Tax amount CHF / cent. from 01.01.2024</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_chtax_303b" model="account.report.line">
                                <field name="name">303b - Standard rate (8,1%): Tax amount CHF / cent. from 01.01.2024</field>
                                <field name="code">tax_ch_303b</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_303b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">303b</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_chtax_313b" model="account.report.line">
                                <field name="name">313b - Reduced rate (2,6%): Tax amount CHF / cent. from 01.01.2024</field>
                                <field name="code">tax_ch_313b</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_313b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">313b</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_chtax_343b" model="account.report.line">
                                <field name="name">343b - Accommodation rate (3,8%): Tax amount CHF / cent. from 01.01.2024</field>
                                <field name="code">tax_ch_343b</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_343b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">343b</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_chtax_383b" model="account.report.line">
                                <field name="name">383b - Acquisition tax: Tax amount CHF / cent. from 01.01.2024</field>
                                <field name="code">tax_ch_383b</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_383b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">383b</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_tax_amount_2" model="account.report.line">
                        <field name="name">Tax amount CHF / cent. to 31.12.2023</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_chtax_302b" model="account.report.line">
                                <field name="name">302b - Standard rate (7,7%): Tax amount CHF / cent. to 31.12.2023</field>
                                <field name="code">tax_ch_302b</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_302b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">302b</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_chtax_312b" model="account.report.line">
                                <field name="name">312b - Reduced rate (2,5%): Tax amount CHF / cent. to 31.12.2023</field>
                                <field name="code">tax_ch_312b</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_312b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">312b</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_chtax_342b" model="account.report.line">
                                <field name="name">342b - Accommodation rate (3,7%): Tax amount CHF / cent. to 31.12.2023</field>
                                <field name="code">tax_ch_342b</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_342b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">342b</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_chtax_382b" model="account.report.line">
                                <field name="name">382b - Acquisition tax: Tax amount CHF / cent. to 31.12.2023</field>
                                <field name="code">tax_ch_382b</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_chtax_382b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">382b</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_399" model="account.report.line">
                        <field name="name">399 - Total amount of tax due</field>
                        <field name="code">tax_ch_399</field>
                        <field name="aggregation_formula">tax_ch_302b.balance + tax_ch_303b.balance + tax_ch_312b.balance + tax_ch_313b.balance + tax_ch_342b.balance + tax_ch_343b.balance + tax_ch_382b.balance + tax_ch_383b.balance</field>
                    </record>
                    <record id="account_tax_report_line_chtax_400" model="account.report.line">
                        <field name="name">400 - Input tax on cost of materials and supplies of services</field>
                        <field name="code">tax_ch_400</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_400_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">400</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_405" model="account.report.line">
                        <field name="name">405 - Input tax on investments and other operating costs</field>
                        <field name="code">tax_ch_405</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_405_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">405</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_410" model="account.report.line">
                        <field name="name">410 - De-taxation (art. 32)</field>
                        <field name="code">tax_ch_410</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_410_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">410</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_415" model="account.report.line">
                        <field name="name">415 - Correction of the input tax deduction: mixed use (art. 30), own use (art. 31)</field>
                        <field name="code">tax_ch_415</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_415_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">415</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_420" model="account.report.line">
                        <field name="name">420 - Reduction of the input tax deduction: Flow of funds, which are not deemed to be consideration, such as subsidies, tourist charges (art. 33 para. 2)</field>
                        <field name="code">tax_ch_420</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_420_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">420</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_479" model="account.report.line">
                        <field name="name">479 - Total Ref. 400 to 420</field>
                        <field name="code">tax_ch_479</field>
                        <field name="aggregation_formula">tax_ch_400.balance + tax_ch_405.balance + tax_ch_410.balance - tax_ch_415.balance - tax_ch_420.balance</field>
                    </record>
                    <record id="account_tax_report_line_chtax_500" model="account.report.line">
                        <field name="name">500 - Amount payable</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_500_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">tax_ch_399.balance - tax_ch_479.balance</field>
                                <field name="subformula">if_above(CHF(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_510" model="account.report.line">
                        <field name="name">510 - Credit in favour of the taxable person</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_510_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">tax_ch_479.balance - tax_ch_399.balance</field>
                                <field name="subformula">if_above(CHF(0))</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_chtax_autres_mouv" model="account.report.line">
                <field name="name">III. OTHER CASH FLOWS</field>
                <field name="hierarchy_level">0</field>
                <field name="aggregation_formula">tax_ch_900.balance + tax_ch_910.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_chtax_900" model="account.report.line">
                        <field name="name">900 - Subsidies, tourist funds collected by tourist offices, contributions from cantonal water, sewage or waste funds (art. 18 para. 2 lit. a to c)</field>
                        <field name="code">tax_ch_900</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_900_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">900</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_chtax_910" model="account.report.line">
                        <field name="name">910 - Donations, dividends, payments of damages etc. (art. 18 para. 2 lit. d to l)</field>
                        <field name="code">tax_ch_910</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_chtax_910_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">910</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-ch.csv

```csv
"id","name","code","account_type","reconcile","name@de","name@it","name@fr","name@ar","name@zh_CN","name@nl"
"ch_coa_1060","Securities (with stock exchange price)","1060","asset_current","False","Wertpapiere (mit Börsenkurs)","Titoli","Titres","","",""
"ch_coa_1069","Accumulated depreciation on securities","1069","asset_current","False","Kumulierte Abschreibungen auf Wertpapiere","Rettifica valore titoli","Ajustement de la valeur des titres","","",""
"ch_coa_1090","Salary-Pass-Through Account","1090","asset_current","True","Gehaltsdurchlaufkonto","Conto di trasferimento dello stipendio","Compte de transfert de salaire","","",""
"ch_coa_1091","Transfer account: Salaries","1091","asset_current","True","Transferkonto: Gehälter","Conto d'attesa - Salari","Compte d'attente pour salaires","","",""
"ch_coa_1099","Transfer account: miscellaneous","1099","asset_current","True","Transferkonto: Verschiedenes","Conto d'attesa - altro","Compte d'attente autre","","",""
"ch_coa_1100","Accounts receivable from goods and services (Debtors)","1100","asset_receivable","True","Forderungen aus Lieferungen und Leistungen (Debitoren)","Crediti da forniture e prestazioni (debitori)","Débiteurs","","",""
"ch_coa_1101","Receivable (PoS)","1101","asset_receivable","True","Forderungen (PoS)","Crediti (Punti Vendita)","Débiteurs (PoS)","","",""
"ch_coa_1109","Del credere (Acc. depr. on debtors)","1109","asset_current","False","Delkredere (Akontoabzug für Schuldner)","Delcredere","Ducroire","","",""
"ch_coa_1140","Advances and loans","1140","asset_current","False","Vorschüsse und Darlehen","Anticipi e prestiti","Avances et prêts","","",""
"ch_coa_1149","Advances and loans adjustments","1149","asset_current","False","Anpassungen von Vorschüssen und Darlehen","Rettifica valore anticipi e prestiti","Ajustement de la valeur des avances et des prêts","","",""
"ch_coa_1170","Input Tax (VAT) receivable on material, goods, services, energy","1170","asset_current","False","Vorsteuer (MwSt.) auf Material, Waren, Dienstleistungen, Energie","IVA, Imposta precedente su materiale, merce, servizi e energia","Impôt préalable: TVA s/matériel, marchandises, prestations et énergie","","",""
"ch_coa_1171","Input Tax (VAT) receivable on investments, other operating expenses","1171","asset_current","False","Vorsteuer (MwSt.) auf Investitionen, sonstige betriebliche Aufwendungen","IVA, imposta precedente su investimenti e altri costi d’esercizio","Impôt préalable: TVA s/investissements et autres charges d’exploitation","","",""
"ch_coa_1176","Withholding Tax (WT) receivable","1176","asset_current","False","Forderungen Quellensteuer","Imposta preventiva","Impôt anticipé","","",""
"ch_coa_1180","Receivables from social insurances and social security institutions","1180","asset_current","False","Forderungen gegenüber Sozialversicherungen und Vorsorgeeinrichtungen","Crediti da assicurazioni sociali e istituti di previdenza","Créances envers les assurances sociales et institutions de prévoyance","","",""
"ch_coa_1189","Withholding tax","1189","asset_current","False","Quellensteuer","Imposte alla fonte","Impôt à la source","","",""
"ch_coa_1190","Other short-term receivables","1190","asset_current","False","Sonstige kurzfristige Forderungen","Altri crediti a breve termine","Autres créances à court terme","","",""
"ch_coa_1199","Accumulated depreciation on short-terms receivables","1199","asset_current","False","Kumulierte Abschreibungen auf kurzfristige Forderungen","Rettifica valore crediti diversi a breve termine","Ajustement de la valeur des créances à court terme","","",""
"ch_coa_1200","Goods / Merchandise (Trade)","1200","asset_current","False","Waren / Güter (Handel)","Merce di rivendita","Marchandises commerciales","","",""
"ch_coa_1207","Accumulated depreciation on Goods / Merchandise (Trade)","1207","asset_current","False","Kumulierte Abschreibungen auf Waren / Handelswaren (Handel)","Variazione delle rimanenze di merci","Variation des stocks de marchandises","","",""
"ch_coa_1208","Downpayment on Goods / Merchandise (Trade)","1208","asset_current","False","Akonto auf Güter / Handelswaren (Handel)","Acconti su beni commerciali","Acomptes sur les marchandises commerciales","","",""
"ch_coa_1209","Correction on Goods / Merchandise (Trade)","1209","asset_current","False","Wertberichtigung Handelswaren","Rettifiche di valore delle scorte di beni","Corrections de la valeur des stocks de marchandises","","",""
"ch_coa_1210","Raw materials","1210","asset_current","False","Rohstoffe","Materie prime","Matières premières","","",""
"ch_coa_1217","Accumulated depreciation on raw material","1217","asset_current","False","Kumulierte Abschreibung von Rohstoffen","Variazione delle rimanenze di materie prime","Variation des stocks des matières premières","","",""
"ch_coa_1218","Downpayment on raw material","1218","asset_current","False","Anzahlung auf Rohmaterial","Acconti su materie prime","Acomptes sur matières premières","","",""
"ch_coa_1219","Correction on raw material","1219","asset_current","False","Berichtigung Rohmaterial","Rettifiche di valore su materie prime","Corrections de la valeur sur matières premières","","",""
"ch_coa_1220","Auxiliary material","1220","asset_current","False","Hilfsmaterial","Materiali ausiliari","Matières auxiliaires","","",""
"ch_coa_1230","Consumables","1230","asset_current","False","Verbrauchsgüter","Materiale di consumo","Matières consommables","","",""
"ch_coa_1250","Consignments Goods","1250","asset_current","False","","","","","",""
"ch_coa_1260","Finished products","1260","asset_current","False","Fertige Erzeugnisse","Prodotti finiti","Stocks de produits finis","","",""
"ch_coa_1267","Accumulated depreciation on Finished products","1267","asset_current","False","Bestandesänderungen fertige Erzeugnisse","Variazione delle scorte di prodotti finiti","Variation de stocks de produits finis","","",""
"ch_coa_1269","Correction on Finished products","1269","asset_current","False","Wertberichtigungen fertige Erzeugnisse","Correzione del valore delle rimanenze di prodotti finiti","Correction de la valeur de stocks de produits finis","","",""
"ch_coa_1270","Products in process / Unfinished products","1270","asset_current","False","Unfertige Erzeugnisse","Scorte di semilavorati","Stocks de produits semi-ouvrés","","",""
"ch_coa_1277","Accumulated depreciation on Products in process / Unfinished products","1277","asset_current","False","Kumulierte Abschreibungen auf Waren in Arbeit / Unfertige Erzeugnisse","Variazione delle scorte di prodotti semilavorati","Variation de stock produits semi-ouvrés","","",""
"ch_coa_1279","Correction on Products in process / Unfinished products","1279","asset_current","False","Berichtigung für Unfertige Erzeugnisse","Correzioni del valore delle rimanenze di semilavorati","Corrections de la valeur des stock produits semi-ouvrés","","",""
"ch_coa_1280","Work in progress","1280","asset_current","False","Nicht fakturierte Dienstleistungen","Prodotti in corso di fabbricazione","Travaux en cours","","",""
"ch_coa_1287","Accumulated depreciation on work in progress","1287","asset_current","False","Kumulierte Abschreibungen auf laufende Aufträge","Variazione di valore dei lavori in corso","Variation de la valeur des travaux en cours","","",""
"ch_coa_1289","Correction on work in progress","1289","asset_current","False","Korrekturen laufende Projekte","Correzioni di valore di lavori in corso","Corrections de la valeur des travaux en cours","","",""
"ch_coa_1300","Accrued revenue and deferred expense (Accounts paid in advance)","1300","asset_current","False","Rechnungsabgrenzungsposten (im Voraus gezahlte Rechnungen)","Costi anticipati","Charges payées d‘avance","","",""
"ch_coa_1301","Deferred expense (Accounts paid in advance)","1301","asset_current","False","Noch nicht erhaltener Ertrag","Ricavi da incassare","Produits à recevoir","","",""
"ch_coa_1400","Long-term securities","1400","asset_current","False","Langfristige Wertpapiere","Titoli a lungo termine","Titres à long terme","","",""
"ch_coa_1409","Accumulated depreciation on long-term securities","1409","asset_current","False","Kumulierte Abschreibungen auf langfristige Wertpapiere","Rettifica valore titoli","Ajustement de la valeur des titres","","",""
"ch_coa_1440","Loan (Asset)","1440","asset_current","False","Darlehen (Vermögenswert)","Prestiti","Prêts","","",""
"ch_coa_1441","Mortgages","1441","asset_current","False","Hypotheken","Prestiti ipotecari","Hypothèques","","",""
"ch_coa_1449","Accumulated depreciation on long term receivables","1449","asset_current","False","Kumulierte Abschreibungen auf langfristige Forderungen","Rettifica valore crediti a lungo termine","Ajustement de la valeur des créances à long terme","","",""
"ch_coa_1480","Participations","1480","asset_current","False","Beteiligungen","Partecipazioni","Participations","المشتركين","参与","Deelnames"
"ch_coa_1489","Accumulated depreciation on participations","1489","asset_current","False","Kumulierte Abschreibungen auf Beteiligungen","Rettifica valore partecipazioni","Ajustement de la valeur des participations","","",""
"ch_coa_1500","Machinery","1500","asset_current","False","Maschinen","Macchine e attrezzature","Machines et appareils","","","Machinerie"
"ch_coa_1509","Accumulated depreciation on machinery","1509","asset_current","False","Kumulierte Abschreibungen auf Maschinen","Ammortamenti su macchinari e attrezzature","Amortissements sur les machines et appareils","","",""
"ch_coa_1510","Equipment","1510","asset_current","False","Ausrüstung","Mobilio e installazioni","Mobilier et installations","المعدات","设备","Apparatuur"
"ch_coa_1519","Accumulated depreciation on equipment","1519","asset_current","False","Kumulierte Abschreibungen auf Ausrüstungen","Ammortamenti su mobili e arredi","Amortissements sur le mobilier et les installations","","",""
"ch_coa_1520","Office Equipment (including Information & Communication Technology)","1520","asset_current","False","Büroausstattung (einschließlich Informations- und Kommunikationstechnologie)","Macchine ufficio, informatica e tecnologia della comunicazione","Machines de bureau, informatique, systèmes de communication","","",""
"ch_coa_1529","Accumulated depreciation on office equipment (incl. ICT)","1529","asset_current","False","Kumulierte Abschreibungen auf Büroausstattung (einschließlich ICT)","Ammortamenti su macchine da ufficio, inf. e sistemi di comunicazione","Amortissements sur les machines de bureau, inf. et syst. comm.","","",""
"ch_coa_1530","Vehicles","1530","asset_current","False","Fahrzeuge","Veicoli","Véhicules","المركبات","车辆","Voertuigen"
"ch_coa_1539","Accumulated depreciation on vehicles","1539","asset_current","False","Kumulierte Abschreibungen auf Fahrzeuge","Ammortamenti su veicoli","Amortissements sur les véhicules","","",""
"ch_coa_1540","Tools","1540","asset_current","False","Werkzeuge","Utensili e apparecchiature","Outillages et appareils","الأدوات","工具","Tools"
"ch_coa_1549","Accumulated depreciation on tools","1549","asset_current","False","Kumulierte Abschreibungen auf Werkzeuge","Ammortamenti su utensili e attrezzature","Amortissements sur les outillages et appareils","","",""
"ch_coa_1550","Warehouse","1550","asset_current","False","Lager","Strutture per il deposito","Installations de stockage","المستودع","仓库","Magazijn"
"ch_coa_1559","Accumulated depreciation on warehouse","1559","asset_current","False","Kumulierte Abschreibungen auf Lagerhäuser","Ammortamenti sui magazzini","Amortissements sur les installations de stockage","","",""
"ch_coa_1570","Equipment and Facilities","1570","asset_current","False","","","","","",""
"ch_coa_1579","Accumulated depreciation on Equipment and Facilities","1579","asset_current","False","","","","","",""
"ch_coa_1590","Other movable tangible assets","1590","asset_current","False","Sonstige bewegliche Sachanlagen","Altre immobilizzazioni materiali","Autres immobilisations corporelles meubles","","",""
"ch_coa_1599","Accumulated depreciation on Other movable tangible assets","1599","asset_current","False","Kumulierte Abschreibungen auf Sonstige bewegliche Sachanlagen","Ammortamenti su altre immobilizzazioni materiali mobiliari","Amortissements sur les autres immobilisations corporelles meubles","","",""
"ch_coa_1600","Real Estate","1600","asset_current","False","Liegenschaften","Immobili aziendali","Immeubles d’exploitation","العقارات","房地产","Ontroerend goed"
"ch_coa_1609","Accumulated depreciation on real estate","1609","asset_current","False","Kumulierte Abschreibungen auf Immobilien","Ammortamenti su immobili in esercizio","Amortissements sur les immeubles d’exploitation","","",""
"ch_coa_1700","Patents, Licences","1700","asset_current","False","Patente, Lizenzen","Patenti, know-how, licenze, diritti e sviluppo","Brevets, know-how, licences, droits, développement","","",""
"ch_coa_1709","Accumulated depreciation on Patents, Licences","1709","asset_current","False","Kumulierte Abschreibungen auf Patente, Lizenzen","Ammortamento di brevetti, know-how, licenze, diritti, dev.","Amortissements sur les brevets, know-how, licences, droits, dév.","","",""
"ch_coa_1770","Goodwill","1770","asset_current","False","Geschäftswert","","Goodwill","","商誉","Goodwill"
"ch_coa_1779","Accumulated depreciation on goodwill","1779","asset_current","False","Kumulierte Abschreibungen auf den Geschäftswert","Rettifica valore goodwill","Ajustement de la valeur des goodwill","","",""
"ch_coa_1850","Non-paid-in share capital","1850","asset_current","False","Nicht eingezahltes Grundkapital","Capitale azionario, capitale sociale, diritti di partecipazione o capitale della fondazione non versati","Capital actions, capital social, droits de participations ou capital de fondation non versés","","",""
"ch_coa_2000","Accounts payable from goods and services (Creditors)","2000","liability_payable","True","Verbindlichkeiten aus Lieferungen und Leistungen (Gläubiger)","Debiti per forniture e prestazioni (creditori)","Créanciers","","",""
"ch_coa_2030","Prepayments received","2030","liability_current","False","Erhaltene Anzahlungen","Acconti ricevuti","Acomptes de clients","","",""
"ch_coa_2100","Bank Overdraft (Bank)","2100","liability_current","False","Überziehungskredit (Bank)","Debiti bancari","Dettes bancaires","","",""
"ch_coa_2120","Leasing bondings","2120","liability_current","False","Leasing Anleihen","Impegni leasing finanziari","Engagements de financement par leasing","","",""
"ch_coa_2140","Other interest-bearing short terms liabilities","2140","liability_current","False","Sonstige verzinsliche kurzfristige Verbindlichkeiten","Altri debiti a breve termine onerosi","Autres dettes à court terme rémunérées","","",""
"ch_coa_2160","Dettes envers l'actionnaire","2160","liability_current","False","Verbindlichkeiten gegenüber dem Aktionär","Debiti verso l'azionista","","","",""
"ch_coa_2200","Sales Tax (VAT) owed","2200","liability_current","False","Geschuldete MwSt. (Umsatzsteuer)","IVA dovuta","TVA due","","",""
"ch_coa_2201","VAT payable","2201","liability_current","False","Zu zahlende MwSt.","IVA, rendiconto","Décompte TVA","","","Te betalen BTW"
"ch_coa_2206","Withholding Tax (WT) owed","2206","liability_current","False","Geschuldete Quellensteuer","Imposta preventiva","Impôt anticipé dû","","",""
"ch_coa_2208","Direct Taxes","2208","liability_current","False","Direkte Steuern","Imposte dirette","Impôts directs","","",""
"ch_coa_2210","Others short term liabilities","2210","liability_current","False","Sonstige kurzfristige Verbindlichkeiten","Altri debiti a breve termine","Autres dettes à court terme","","",""
"ch_coa_2261","Dividend payouts resolved (Dividends)","2261","liability_current","False","Beschlossene Dividendenausschüttungen (Dividende)","Dividendi","Dividendes","","",""
"ch_coa_2270","Social insurances owed","2270","liability_current","False","Geschuldete Sozialversicherungen","Assicurazioni sociali e istituti di previdenza","Assurances sociales et institutions de prévoyance","","",""
"ch_coa_2271","Retirement Provision","2271","liability_current","False","Altersvorsorge","Previdenza pensionistica","Prévoyance vieillesse","","",""
"ch_coa_2272","Social Insurance CAF/FAK","2272","liability_current","False","Sozialversicherung FAK","assicurazione sociale FAK","Assurances sociales CAF","","",""
"ch_coa_2273","Accident Insurance","2273","liability_current","False","Unfallversicherung","Assicurazione contro gli infortuni","Assurance Accident","","",""
"ch_coa_2274","Daily Sickness Insurance","2274","liability_current","False","Krankentagegeldversicherung","Assicurazione malattia giornaliera","Assurance maladie journalière","","",""
"ch_coa_2279","Withholding taxes","2279","liability_current","False","Quellensteuer","Imposte alla fonte","Impôt à la source","","",""
"ch_coa_2300","Deferred revenue and accrued expenses (Accounts received in advance)","2300","liability_current","False","Rechnungsabgrenzungsposten (im Voraus erhaltene Rechnungen)","Costi da pagare","Charges à payer","","",""
"ch_coa_2301","Deferred revenue (Accounts Received in Advance)","2301","liability_current","False","Umsatzabgrenzungsposten (im Voraus erhaltene Rechnungen)","Ricavi incassati dell’anno seguente","Produits encaissés d’avance","","",""
"ch_coa_2330","Short-term provisions","2330","liability_current","False","Kurzfristige Rückstellungen","Accantonamenti a breve termine","Provisions à court terme","","",""
"ch_coa_2400","Bank debts","2400","liability_current","False","Bankschulden","Debiti bancari","Dettes bancaires","","",""
"ch_coa_2420","Finance lease commitments","2420","liability_current","False","Verpflichtungen aus Finanzierungsleasing","Impegni leasing finanziari","Engagements de financement par leasing","","",""
"ch_coa_2430","Debentures","2430","liability_current","False","Schuldverschreibungen","Prestiti obbligazionari","Emprunts obligataires","","",""
"ch_coa_2450","Loans","2450","liability_current","False","Darlehen","Prestiti","Emprunts","","",""
"ch_coa_2451","Mortgages","2451","liability_current","False","Hypotheken","Prestiti ipotecari","Hypothèques","","",""
"ch_coa_2500","Other long term liabilities","2500","liability_current","False","Sonstige langfristige Verbindlichkeiten","Altri debiti a lungo termine","Autres dettes à long terme","","",""
"ch_coa_2600","Long-term provisions","2600","liability_current","False","Langfristige Rückstellungen","Accantonamenti","Provisions","","",""
"ch_coa_2800","Share capital","2800","equity","False","Grundkapital","Capitale azionario, capitale sociale, diritti di partecipazione o capitale della fondazione","Capital-actions, capital social, capital de fondation","","",""
"ch_coa_2900","Legal capital reserves","2900","equity","False","Gesetzliche Kapitalrücklagen","Riserva legale da capitale","Réserves légales issues du capital","","",""
"ch_coa_2940","Valuation Reserves","2940","equity","False","Bewertungsreserven","Riserve da rivalutazioni","Réserves d‘évaluation","","",""
"ch_coa_2950","Legal retained earnings (Reserves)","2950","equity","False","Gesetzliche Gewinnrücklagen (Reserven)","Riserva legale da utili","Réserves légales issues du bénéfice","","",""
"ch_coa_2960","Voluntary retained earnings","2960","equity","False","Freiwillige Gewinnrücklagen","Riserve facoltative da utili","Réserves libres","","",""
"ch_coa_2970","Profits brought forward / Losses brought forward","2970","equity","False","Gewinnvortrag / Verlustvortrag","Utile / perdita riportata","Bénéfice / perte reporté","","",""
"ch_coa_2979","Annual profit or annual loss","2979","equity","False","Jahresgewinn oder Jahresverlust","Utile/perdita annuale","Bénéfice / perte de l’exercice","","",""
"ch_coa_2980","Treasury stock, shares, participation rights (negative item)","2980","equity","False","","","","","",""
"ch_coa_3000","Sales of products (Manufacturing)","3000","income","False","Verkauf von Produkten (Herstellung)","Ricavi prodotti fabbricati","Ventes de produits fabriqués","","",""
"ch_coa_3009","Deductions on sales","3009","income","False","Abzüge bei Verkäufen","Diminuzione di ricavi","Déductions sur ventes","","",""
"ch_coa_3200","Sales of goods (Trade)","3200","income","False","Warenverkauf (Handel)","Ricavi merci di rivendita","Ventes de marchandises","","",""
"ch_coa_3400","Revenues from services","3400","income","False","Erlöse aus Dienstleistungen","Ricavi prestazioni di servizi","Ventes de prestations","","",""
"ch_coa_3600","Other revenues","3600","income","False","Sonstige Einnahmen","Altri ricavi e prestazioni di servizi","Autres ventes et prestations de services","","",""
"ch_coa_3700","Own services","3700","income","False","Eigene Leistungen","Lavori interni","Prestations propres","","",""
"ch_coa_3710","Own consumption","3710","income","False","Eigenverbrauch","Consumo proprio","Consommations propres","","",""
"ch_coa_3800","Financial discount","3800","income","False","Erlösminderung","Sconti","Escomptes","","",""
"ch_coa_3801","Discounts and price reduction","3801","income","False","Rabatte und Preisnachlässe","Sconti e riduzioni di prezzo","Rabais et réduction de prix","","",""
"ch_coa_3802","Rebates","3802","income","False","Nachlässe","Sconti","Ristournes","","",""
"ch_coa_3803","Third-party commissions","3803","income","False","Provisionen Dritter","Commissioni di terzi","Commissions de tiers","","",""
"ch_coa_3804","Collection fees","3804","income","False","Inkassogebühren","Tasse di riscossione","Frais d'encaissement","","",""
"ch_coa_3805","Losses from bad debts","3805","income","False","Verluste aus Forderungsausfällen","Perdite su crediti commerciali, variazione del fondo svalutazione crediti","Pertes sur créances clients, variation ducroire","","",""
"ch_coa_3806","Exchange rate differences","3806","income","False","Wechselkursdifferenzen","Differenze di cambio","Différences de change","","",""
"ch_coa_3807","Shipping & Returns","3807","income","False","Versand und Rücksendung","Costi di spedizione","Frais d'expédition","","",""
"ch_coa_3900","Changes in inventories of unfinished and finished products","3900","income","False","Veränderung des Bestandes unfertiger und fertiger Erzeugnisse","Variazione delle scorte di prodotti in corso di fabbricazione","Variation des stocks de produits semi-finis","","",""
"ch_coa_3901","Change in inventories of finished goods","3901","income","False","Veränderung des Bestandes an Fertigerzeugnissen","Variazione delle scorte di prodotti finiti","Variation des stocks de produits finis","","",""
"ch_coa_3940","Change in the value of unbilled services","3940","income","False","Bestandesänderungen nicht fakturierte Dienstleistungen","Variazione prestazioni di servizi non fatturate","Variation de la valeur des prestations non facturées","","",""
"ch_coa_4000","Cost of raw materials (Manufacturing)","4000","expense","False","Materialaufwand Produktion","Costi materiale per la fabbricazione","Charges de matériel de l‘atelier","","",""
"ch_coa_4008","Inventory changes","4008","expense","False","Bestandsänderungen","Variazioni delle rimanenze","Variations de stocks","","",""
"ch_coa_4009","Deductions obtained on purchases","4009","expense","False","Bei Käufen erzielte Abzüge","Deduzioni ottenute sugli acquisti","Déductions obtenues sur achats","","",""
"ch_coa_4070","Purchase Loans","4070","expense","False","Frachtkosten","Acquisto di prestiti","Frêts à l'achat","","",""
"ch_coa_4071","Customs duties on importation","4071","expense","False","Einfuhrzölle","Dazi doganali all'importazione","Droits de douanes à l'importation","","",""
"ch_coa_4072","Transport costs at purchase","4072","expense","False","Transportkosten beim Erwerb","Costi di trasporto all'acquisto","Frais de transport à l'achat","","",""
"ch_coa_4080","Inventory changes","4080","expense","False","Bestandsänderungen","Variazioni delle rimanenze","Variations de stocks","","",""
"ch_coa_4086","Loss of material","4086","expense","False","Materialverlust","Perdita di materiale","Pertes de matières","","",""
"ch_coa_4200","Cost of materials (Trade)","4200","expense","False","Materialkosten (Handel)","Acquisti di beni destinati alla rivendita","Achats de marchandises destinées à la revente","","",""
"ch_coa_4400","Cost of purchased services","4400","expense","False","Aufwand für bezogene Dienstleistungen","Lavori di terzi / prestazioni di subappaltanti","Prestations / travaux de tiers","","",""
"ch_coa_4500","Electricity","4500","expense","False","Elektrizität","Elettricità","Electricité","","",""
"ch_coa_4510","Gas","4510","expense","False","Gas","","Gaz","","",""
"ch_coa_4520","Fuel oil","4520","expense","False","Heizöl","Olio combustibile","Mazout","","",""
"ch_coa_4521","Coal, briquettes, wood","4521","expense","False","Kohle, Briketts, Holz","Carbone, bricchette, legno","Charbon, briquettes, bois","","",""
"ch_coa_4530","Petrol","4530","expense","False","Benzin","Carburante","Essence","","",""
"ch_coa_4540","Water","4540","expense","False","Wasser","Acqua","Eau","ماء","水","Water"
"ch_coa_4800","Change in inventories of goods","4800","expense","False","Veränderung der Warenvorräte","Variazione delle rimanenze di merci","Variation des stocks de marchandises","","",""
"ch_coa_4801","Change in raw material inventories","4801","expense","False","Änderung der Rohstoffvorräte","Variazione delle rimanenze di materie prime","Variation des stocks de matières premières","","",""
"ch_coa_4900","Financial Discounts","4900","expense","False","Finanzielle Ermäßigungen","Sconti","Escomptes","","",""
"ch_coa_4901","Discounts and price reductions","4901","expense","False","Rabatte und Preisnachlässe","Sconti e riduzioni di prezzo","Rabais et réductions de prix","","",""
"ch_coa_4092","Rebates","4902","expense","False","Nachlässe","Sconti","Ristournes","","",""
"ch_coa_4903","Commissions on purchases","4903","expense","False","Provisionen auf Käufe","Commissioni sugli acquisti","Commissions obtenues sur achats","","",""
"ch_coa_4906","Exchange rate differences","4906","expense","False","Wechselkursdifferenzen","Differenze di cambio","Différences de change","","",""
"ch_coa_4991","Cash Difference Loss","4991","expense","False","Bardifferenzverlust","Perdita per Differenza di Cassa","Perte de Différence de change","خسارة الفرق النقدي","
现金差额损失","Verlies door Kasverschil"
"ch_coa_4992","Cash Difference Gain","4992","income_other","False","Bardifferenzgewinn","Utile da Differenza di Cassa","Gain de différence de change","مكاسب الفرق النقدي","现金差额收益","Winst uit Kasverschil"
"ch_coa_5000","Wages and salaries","5000","expense","False","Löhne und Gehälter","Salari","Salaires","","",""
"ch_coa_5001","Social Insurance Payments","5001","expense","False","Sozialversicherungsbeiträge","Pagamenti dell'assicurazione sociale","Paiements d'assurance sociale","","",""
"ch_coa_5002","Profit Sharing","5002","expense","False","Gewinnbeteiligung","Partecipazione agli utili","Participation aux bénéfices","","",""
"ch_coa_5003","Salary Advance","5003","expense","False","","","","","",""
"ch_coa_5004","Holidays payment after departure","5004","expense","False","","","","","",""
"ch_coa_5005","13th Month","5005","expense","False","","","","","",""
"ch_coa_5006","Gratification","5006","expense","False","","","","","",""
"ch_coa_5007","Bonus","5007","expense","False","","","","","",""
"ch_coa_5009","Jubilee Gift","5009","expense","False","","","","","",""
"ch_coa_5010","Commissions","5010","expense","False","","","","","",""
"ch_coa_5030","Travelling Expenses","5030","expense","False","","","","","",""
"ch_coa_5031","Company Car Correction","5031","expense","False","","","","","",""
"ch_coa_5040","Salary Allowances","5040","expense","False","","","","","",""
"ch_coa_5601","CA Fees","5601","expense","False","","","","","",""
"ch_coa_5700","Social benefits","5700","expense","False","Sozialleistungen","Oneri sociali","Charges sociales","","",""
"ch_coa_5701","Social benefits AC","5701","expense","False","","","","","",""
"ch_coa_5710","Family compensation fund","5710","expense","False","Familienausgleichskasse","Fondo di compensazione familiare","Caisse de compensation familiale","","",""
"ch_coa_5720","Social Benefits LPP","5720","expense","False","","","","","",""
"ch_coa_5721","Social Benefits LPP Redemption","5721","expense","False","","","","","",""
"ch_coa_5730","Social Benefits AANP","5730","expense","False","","","","","",""
"ch_coa_5731","Social Benefits Optional LAAC Redemption","5731","expense","False","","","","","",""
"ch_coa_5740","Social Benefits IJM/CM","5740","expense","False","","","","","",""
"ch_coa_5790","Withholding Taxes IS","5790","liability_current","False","","","","","",""
"ch_coa_5800","Other staff cost","5800","expense","False","Sonstige Personalkosten","Altri costi del personale","Autres charges du personnel","","",""
"ch_coa_5810","Professional Training","5810","expense","False","Berufsausbildung","Formazione professionale","Formation Professionnelle","","",""
"ch_coa_5820","Travel Expenses","5820","expense","False","","","","","",""
"ch_coa_5821","Lunch Expenses","5821","expense","False","","","","","",""
"ch_coa_5822","Nightly Expenses","5822","expense","False","","","","","",""
"ch_coa_5830","Representation Fees","5830","expense","False","","","","","",""
"ch_coa_5831","Car Fees","5831","expense","False","","","","","",""
"ch_coa_5832","Other Fees","5832","expense","False","","","","","",""
"ch_coa_5840","Indemnities","5840","expense","False","","","","","",""
"ch_coa_5890","Rate of personal use","5890","expense","False","","","","","",""
"ch_coa_5891","Compensation company car use","5891","expense","False","","","","","",""
"ch_coa_5900","Temporary staff expenditures","5900","expense","False","Leistungen DritterAusgaben für Zeitarbeitskräfte","Spese per il personale temporaneo","Charges de personnels temporaires","","",""
"ch_coa_6000","Rent","6000","expense","False","Miete","Costi dei locali","Charges de locaux","","",""
"ch_coa_6100","Maintenance & repair expenses","6100","expense","False","Kosten für Wartung und Reparatur","Manutenzioni, riparazioni e sostituzione immobilizzazioni mobiliari","Entretien, réparations et remplacement des inst. servant à l’exploitation","","",""
"ch_coa_6105","Leasing movable tangible fixed assets","6105","expense","False","Leasing von beweglichen Sachanlagen","Leasing di immobilizzazioni materiali mobiliari","Leasing immobilisations corporelles meubles","","",""
"ch_coa_6200","Vehicle expenses","6200","expense","False","Fahrzeugkosten","Costi auto e di trasporto","Charges de véhicules et de transport","","",""
"ch_coa_6260","Vehicules leasing and renting","6260","expense","False","Fahrzeugleasing und -vermietung","Leasing e noleggio auto","Leasing et location de véhicules","","",""
"ch_coa_6300","Insurance premiums","6300","expense","False","Versicherungsprämien","Assicurazioni - dazi, tasse, autorizzazioni","Assurances-choses, droits, taxes, autorisations","","",""
"ch_coa_6400","Energy expenses & disposal expenses","6400","expense","False","Energie- und Entsorgungsaufwand","Costi energia e smaltimento","Charges d’énergie et évacuation des déchets","","",""
"ch_coa_6500","Administration expenses","6500","expense","False","Verwaltungskosten","Costi amministrativi","Charges d‘administration","","",""
"ch_coa_6570","IT leasing","6570","expense","False","IT Leasing","Costi informatici incluso leasing","Charges et leasing d’informatique","","",""
"ch_coa_6600","Promotion and advertising expenses","6600","expense","False","Werbeaufwand","Costi pubblicitari","Publicité","","",""
"ch_coa_6700","Other operating expenses","6700","expense","False","Sonstige betriebliche Aufwendungen","Altri costi d’esercizio","Autres charges d‘exploitation","","",""
"ch_coa_6800","Depreciations","6800","expense","False","Abschreibung","Ammortamenti e rettifiche di valore dell’attivo fisso","Amortissements et ajustements de valeur des postes sur immobilisations corporelles","","",""
"ch_coa_6900","Financial expenses (Interest expenses, Securities expenses, Participations expenses)","6900","expense","False","Finanzaufwand (Zinsaufwand, Wertpapieraufwand, Beteiligungsaufwand)","Costi finanziari","Charges financières","","",""
"ch_coa_6950","Financial revenues (Interest revenues, Securities revenues, Participations revenues)","6950","expense","False","Finanzerträge (Zinserträge, Wertpapiererträge, Beteiligungserträge)","Ricavi finanziari","Produits financiers","","",""
"ch_coa_7000","Non-core business revenues","7000","income","False","Erträge aus Nicht-Kerngeschäft","Ricavi attività accessoria","Produits accessoires","","",""
"ch_coa_7010","Non-core business expenses","7010","expense","False","Nicht zum Kerngeschäft gehörende Ausgaben","Costi attività accessoria","Charges accessoires","","",""
"ch_coa_7500","Revenues from operational real estate","7500","income","False","Erlöse aus betrieblichen Liegenschaften","Ricavi immobili aziendali","Produits des immeubles d‘exploitation","","",""
"ch_coa_7510","Expenses from operational real estate","7510","expense","False","Aufwand betriebliche Liegenschaft","Costi immobili aziendali","Charges des immeubles d‘exploitation","","",""
"ch_coa_8000","Non-operational expenses","8000","expense","False","Nichtbetriebliche Aufwendungen","Costi estranei","Charges hors exploitation","","",""
"ch_coa_8100","Non-operational revenues","8100","income","False","Betriebsfremde Erträge","Ricavi estranei","Produits hors exploitation","","",""
"ch_coa_8500","Extraordinary expenses","8500","expense","False","Außerordentliche Ausgaben","Costi straordinari, unici o relativi ad altri periodi contabili","Charges extraordinaires, exceptionnelles ou hors période","","",""
"ch_coa_8510","Extraordinary revenues","8510","income","False","Außerordentliche Ausgaben","Ricavi straordinari, unici o relativi ad altri periodi contabili","Produits extraordinaires, exceptionnels ou hors période","","",""
"ch_coa_8900","Direct Taxes","8900","expense","False","Direkte Steuern","Imposte dirette","Impôts directs","","",""

```

## File: data\template\account.fiscal.position-ch.csv

```csv
"id","name","auto_apply","country_group_id","sequence","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@de"
"fiscal_position_template_1","Suisse national (+Liechtenstein)","1","base.ch_and_li","","","",""
"fiscal_position_template_import","Import/Export","1","","1","vat_25_purchase","vat_O_import","Import/Export"
"","","","","","vat_25_invest","vat_O_import",""
"","","","","","vat_37_purchase","vat_O_import",""
"","","","","","vat_37_invest","vat_O_import",""
"","","","","","vat_77_purchase_reverse","vat_O_import",""
"","","","","","vat_77_invest","vat_O_import",""
"","","","","","vat_25","vat_XO",""
"","","","","","vat_37","vat_XO",""
"","","","","","vat_77","vat_XO",""
"","","","","","vat_purchase_26","vat_O_import",""
"","","","","","vat_purchase_26_invest","vat_O_import",""
"","","","","","vat_purchase_38","vat_O_import",""
"","","","","","vat_purchase_38_invest","vat_O_import",""
"","","","","","vat_purchase_81_reverse","vat_O_import",""
"","","","","","vat_purchase_81_invest","vat_O_import",""
"","","","","","vat_sale_26","vat_XO",""
"","","","","","vat_sale_38","vat_XO",""
"","","","","","vat_sale_81","vat_XO",""

```

## File: data\template\account.tax-ch.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","price_include","sequence","active","children_tax_ids","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@de","description@it","description@fr"
"vat_25","2.5%","","2.5%","2.5","percent","sale","tax_group_tva_25","","","False","","base","invoice","+312a","","","",""
"","","","","","","","","","","","","tax","invoice","+312b","ch_coa_2200","","",""
"","","","","","","","","","","","","base","refund","-312a","","","",""
"","","","","","","","","","","","","tax","refund","-312b","ch_coa_2200","","",""
"vat_25_incl","2.5% INC","2.5% Sales (incl.)","2.5% incl.","2.5","percent","sale","tax_group_tva_25","True","","False","","base","invoice","+312a","","UST 2.5% Lief./DL (inkl. MWST)","IVA dovuta al 2,5% (Incl. TR)","TVA due à 2.5% (Incl. TR)"
"","","","","","","","","","","","","tax","invoice","+312b","ch_coa_2200","","",""
"","","","","","","","","","","","","base","refund","-312a","","","",""
"","","","","","","","","","","","","tax","refund","-312b","ch_coa_2200","","",""
"vat_25_purchase","2.5%","","2.5%","2.5","percent","purchase","tax_group_tva_25","","","False","","base","invoice","","","","",""
"","","","","","","","","","","","","tax","invoice","+400","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-400","ch_coa_1170","","",""
"vat_25_purchase_incl","2.5% INC","2.5% on goods and services (incl.)","2.5% incl.","2.5","percent","purchase","tax_group_tva_25","True","","False","","base","invoice","","","VST 2.5% Mat.-/DL (inkl. MWST)","IVA 2,5% sull'acquisto di B&S (Incl. TR)","TVA 2.5% sur achat B&S (Incl. TR)"
"","","","","","","","","","","","","tax","invoice","+400","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-400","ch_coa_1170","","",""
"vat_25_invest","2.5% I OE","2.5% on invest. and others expenses","2.5% invest.","2.5","percent","purchase","tax_group_tva_25","","","False","","base","invoice","","","VST 2.5% Inv./übr.BA (exkl. MWST)","IVA 2.5% Investimenti e altri costi (TR)","TVA 2.5% sur invest. et autres ch. (TR)"
"","","","","","","","","","","","","tax","invoice","+405","ch_coa_1171","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-405","ch_coa_1171","","",""
"vat_25_invest_incl","2.5% I OE INC","2.5% on invest. and others expenses (incl.)","2.5% invest. incl.","2.5","percent","purchase","tax_group_tva_25","True","","False","","base","invoice","","","VST 2.5% Inv./übr.BA (inkl. MWST)","IVA 2,5% su investimenti e altre voci (incl. TR)","TVA 2.5% sur invest. et autres ch. (Incl. TR)"
"","","","","","","","","","","","","tax","invoice","+405","ch_coa_1171","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-405","ch_coa_1171","","",""
"vat_37","3.7%","","3.7%","3.7","percent","sale","tax_group_tva_37","","","False","","base","invoice","+342a","","","",""
"","","","","","","","","","","","","tax","invoice","+342b","ch_coa_2200","","",""
"","","","","","","","","","","","","base","refund","-342a","","","",""
"","","","","","","","","","","","","tax","refund","-342b","ch_coa_2200","","",""
"vat_37_incl","3.7% INC","3.7% Sales (incl.)","3.7% incl.","3.7","percent","sale","tax_group_tva_37","True","","False","","base","invoice","+342a","","UST 3.7% Lief./DL (inkl. MWST)","IVA dovuta al 3,7% (Incl. TS)","TVA due à 3.7% (Incl. TS)"
"","","","","","","","","","","","","tax","invoice","+342b","ch_coa_2200","","",""
"","","","","","","","","","","","","base","refund","-342a","","","",""
"","","","","","","","","","","","","tax","refund","-342b","ch_coa_2200","","",""
"vat_37_purchase","3.7%","","3.7%","3.7","percent","purchase","tax_group_tva_37","","","False","","base","invoice","","","","",""
"","","","","","","","","","","","","tax","invoice","+400","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-400","ch_coa_1170","","",""
"vat_37_purchase_incl","3.7% INC","3.7% on goods and services (incl.)","3.7% incl.","3.7","percent","purchase","tax_group_tva_37","True","","False","","base","invoice","","","VST 3.7% Mat.-/DL (inkl. MWST)","IVA 3,7% sull'acquisto di B&S (Incl. TS)","TVA 3.7% sur achat B&S (Incl. TS)"
"","","","","","","","","","","","","tax","invoice","+400","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-400","ch_coa_1170","","",""
"vat_37_invest","3.7% I OE","3.7% on invest. and others expenses","3.7% invest.","3.7","percent","purchase","tax_group_tva_37","","","False","","base","invoice","","","VST 3.7%  Inv./übr.BA (exkl. MWST)","IVA 3,7% su investimenti e altre voci (TS)","TVA 3.7% sur invest. et autres ch. (TS)"
"","","","","","","","","","","","","tax","invoice","+405","ch_coa_1171","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-405","ch_coa_1171","","",""
"vat_37_invest_incl","3.7% I OE INC","3.7% on invest. and others expenses (incl.)","3.7% invest. incl.","3.7","percent","purchase","tax_group_tva_37","True","","False","","base","invoice","","","VST 3.7%  Inv./übr.BA (inkl. MWST)","IVA 3,7% su investimenti e altre voci (incl. TS)","TVA 3.7% sur invest. et autres ch. (Incl. TS)"
"","","","","","","","","","","","","tax","invoice","+405","ch_coa_1171","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-405","ch_coa_1171","","",""
"vat_77","7.7%","","7.7%","7.7","percent","sale","tax_group_tva_77","","0","False","","base","invoice","+302a","","","",""
"","","","","","","","","","","","","tax","invoice","+302b","ch_coa_2200","","",""
"","","","","","","","","","","","","base","refund","-302a","","","",""
"","","","","","","","","","","","","tax","refund","-302b","ch_coa_2200","","",""
"vat_77_incl","7.7% INC","7.7% Sales (incl.)","7.7% incl.","7.7","percent","sale","tax_group_tva_77","True","0","False","","base","invoice","+302a","","UST 7.7% Lief./DL (inkl. MWST)","IVA dovuta al 7,7% (Incl. TN)","TVA due à 7.7% (Incl. TN)"
"","","","","","","","","","","","","tax","invoice","+302b","ch_coa_2200","","",""
"","","","","","","","","","","","","base","refund","-302a","","","",""
"","","","","","","","","","","","","tax","refund","-302b","ch_coa_2200","","",""
"vat_77_purchase_incl","7.7% INC","7.7% on goods and services (incl.)","7.7% incl.","7.7","percent","purchase","tax_group_tva_77","True","0","False","","base","invoice","","","VST 7.7% Mat.-/DL (inkl. MWST)","IVA 7,7% sull'acquisto di B&S (Incl. TN)","TVA 7.7% sur achat B&S (Incl. TN)"
"","","","","","","","","","","","","tax","invoice","+400","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-400","ch_coa_1170","","",""
"vat_77_invest","7.7% I OE","7.7% on invest. and others expenses","7.7% invest.","7.7","percent","purchase","tax_group_tva_77","","","False","","base","invoice","","","VST 7.7% Inv./übr.BA (exkl. MWST)","IVA 7,7% su investimenti e altre voci (TN)","TVA 7.7% sur invest. et autres ch. (TN)"
"","","","","","","","","","","","","tax","invoice","+405","ch_coa_1171","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-405","ch_coa_1171","","",""
"vat_77_invest_incl","7.7% I OE INC","7.7% on invest. and others expenses (incl.)","7.7% invest. incl.","7.7","percent","purchase","tax_group_tva_77","True","","False","","base","invoice","","","VST 7.7% Inv./übr.BA (inkl. MWST)","IVA 7,7% su investimenti e altre voci (incl. TN)","TVA 7.7% sur invest. et autres ch. (Incl. TN)"
"","","","","","","","","","","","","tax","invoice","+405","ch_coa_1171","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-405","ch_coa_1171","","",""
"vat_XO","0% EX","Export","0%","0.0","percent","sale","tax_group_tva_0","","","True","","base","invoice","+220","","0% Export","IVA dovuta 0% (Export)","TVA due a 0% (Exportations)"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","-220","","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"vat_O_exclude","0% EXC","Exclude","0% excl.","0.0","percent","sale","tax_group_tva_0","","","True","","base","invoice","+230","","0% Ausgenommen","IVA 0% Esclusa","TVA 0% exclue"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","-230","","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"vat_O_import","0% EX","Import","0% import","0.0","percent","purchase","tax_group_tva_0","","","True","","base","invoice","","","0% Import","IVA 0% Importazioni di bene e servizi","TVA 0% Importations de biens et services"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"vat_100_import","100% GS","Customs VAT on goods and services","100% imp.","100.0","division","purchase","tax_group_tva_100","True","","True","","base","invoice","","","Zoll Mehrwertsteuer auf Waren und Dienstleistungen","Liquidazione IVA al 100%",""
"","","","","","","","","","","","","tax","invoice","+400","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-400","ch_coa_1170","","",""
"vat_100_import_invest","100% I OE","Customs VAT on invest. and others expenses","100% imp. invest.","100.0","division","purchase","tax_group_tva_100","True","","True","","base","invoice","","","Zoll Mehrwertsteuer auf Investitionen und andere Ausgaben","100 % iva dogana","Dédouanement TVA (invest. et autres ch.)"
"","","","","","","","","","","","","tax","invoice","+405","ch_coa_1171","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-405","ch_coa_1171","","",""
"vat_77_purchase_return","7.7% R","7.7% Sales (reverse)","7.7% return","-7.7","percent","none","tax_group_tva_77","","0","False","","base","invoice","-382a","","UST 7.7% Bezugssteuer","IVA dovuta al 7,7% (TN) (rendimento)","TVA due a 7.7% (TN) (return)"
"","","","","","","","","","","","","tax","invoice","-382b","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","+382a","","","",""
"","","","","","","","","","","","","tax","refund","+382b","ch_coa_1170","","",""
"vat_77_purchase","7.7%","","7.7%","7.7","percent","purchase","tax_group_tva_77","","0","False","","base","invoice","","","","",""
"","","","","","","","","","","","","tax","invoice","+400","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-400","ch_coa_1170","","",""
"vat_77_purchase_reverse","7.7% R C","7.7% on purchase of service abroad (reverse charge)","7.7% reverse","","group","purchase","tax_group_tva_77","","","False","vat_77_purchase_return,vat_77_purchase","","","","","BZS 7.7% Bezugssteuer","IVA 7,7% sull'acquisto di servizi all'estero (reverse charge)","TVA 7.7% sur achat service a l'etranger (reverse charge)"
"vat_other_movements_900","0% S T","0% - Subsidies, tourist taxes","0% subventions","0.0","percent","sale","tax_group_tva_0","","","True","","base","invoice","+900","","0% - Subventionen, Kurtaxen","Sovvenzioni, 0% tasse turistiche","Subventions, taxes touristiques à 0%"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","-900","","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"vat_other_movements_910","0% D","0% - Donations, dividends, compensation","0% dons","0.0","percent","sale","tax_group_tva_0","","","True","","base","invoice","+910","","0% - Schenkungen, Dividenden, Entschädigungen","Donazioni, dividendi, compensi a 0%","Dons, dividendes, dédommagements à 0%"
"","","","","","","","","","","","","tax","invoice","","","","",""
"","","","","","","","","","","","","base","refund","-910","","","",""
"","","","","","","","","","","","","tax","refund","","","","",""
"vat_sale_26","2.6%","2.6% Sales","2.6%","2.6","percent","sale","tax_group_vat_26","","","True","","base","invoice","+313a","","UST 2,6% Lief./DL (exkl. MWST)","IVA dovuta al 2,6% (TR)","TVA due à 2,6% (TR)"
"","","","","","","","","","","","","tax","invoice","+313b","ch_coa_2200","","",""
"","","","","","","","","","","","","base","refund","-313a","","","",""
"","","","","","","","","","","","","tax","refund","-313b","ch_coa_2200","","",""
"vat_sale_26_incl","2.6% INC","2.6% Sales (incl.)","2.6% incl.","2.6","percent","sale","tax_group_vat_26","True","","True","","base","invoice","+313a","","UST 2,6% Lief./DL (inkl. MWST)","IVA dovuta al 2,6% (Incl. TR)","TVA due à 2,6% (Incl. TR)"
"","","","","","","","","","","","","tax","invoice","+313b","ch_coa_2200","","",""
"","","","","","","","","","","","","base","refund","-313a","","","",""
"","","","","","","","","","","","","tax","refund","-313b","ch_coa_2200","","",""
"vat_purchase_26","2.6%","2.6% on goods and services","2.6%","2.6","percent","purchase","tax_group_vat_26","","","True","","base","invoice","","","VST 2,6% Mat.-/DL (exkl. MWST)","IVA 2,6% sull'acquisto di B&S (TR)","TVA 2,6% sur achat B&S (TR)"
"","","","","","","","","","","","","tax","invoice","+400","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-400","ch_coa_1170","","",""
"vat_purchase_26_incl","2.6% INC","2.6% on goods and services (incl.)","2.6% incl.","2.6","percent","purchase","tax_group_vat_26","True","","True","","base","invoice","","","VST 2,6% Mat.-/DL (inkl. MWST)","IVA 2,6% sull'acquisto di B&S (Incl. TR)","TVA 2,6% sur achat B&S (Incl. TR)"
"","","","","","","","","","","","","tax","invoice","+400","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-400","ch_coa_1170","","",""
"vat_purchase_26_invest","2.6% I OE","2.6% on invest. and others expenses","2.6% invest.","2.6","percent","purchase","tax_group_vat_26","","","True","","base","invoice","","","VST 2,6% Inv./übr.BA (exkl. MWST)","IVA 2,6% Investimenti e altri costi (TR)","TVA 2,6% sur invest. et autres ch. (TR)"
"","","","","","","","","","","","","tax","invoice","+405","ch_coa_1171","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-405","ch_coa_1171","","",""
"vat_purchase_26_invest_incl","2.6% I OE INC","2.6% on invest. and others expenses (incl.)","2.6% invest. incl.","2.6","percent","purchase","tax_group_vat_26","True","","True","","base","invoice","","","VST 2,6% Inv./übr.BA (inkl. MWST)","IVA 2,6% su investimenti e altre voci (incl. TR)","TVA 2,6% sur invest. et autres ch. (Incl. TR)"
"","","","","","","","","","","","","tax","invoice","+405","ch_coa_1171","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-405","ch_coa_1171","","",""
"vat_sale_38","3.8%","3.8% Sales","3.8%","3.8","percent","sale","tax_group_vat_38","","","True","","base","invoice","+343a","","UST 3,8% Lief./DL (exkl. MWST)","IVA dovuta al 3,8% (TS)","TVA due à 3,8% (TS)"
"","","","","","","","","","","","","tax","invoice","+343b","ch_coa_2200","","",""
"","","","","","","","","","","","","base","refund","-343a","","","",""
"","","","","","","","","","","","","tax","refund","-343b","ch_coa_2200","","",""
"vat_sale_38_incl","3.8% INC","3.8% Sales (incl.)","3.8% incl.","3.8","percent","sale","tax_group_vat_38","True","","True","","base","invoice","+343a","","UST 3,8% Lief./DL (inkl. MWST)","IVA dovuta al 3,8% (Incl. TS)","TVA due à 3,8% (Incl. TS)"
"","","","","","","","","","","","","tax","invoice","+343b","ch_coa_2200","","",""
"","","","","","","","","","","","","base","refund","-343a","","","",""
"","","","","","","","","","","","","tax","refund","-343b","ch_coa_2200","","",""
"vat_purchase_38","3.8%","3.8% purchase on goods and services","3.8%","3.8","percent","purchase","tax_group_vat_38","","","True","","base","invoice","","","VST 3,8% Mat.-/DL (exkl. MWST)","IVA 3,8% sull'acquisto di B&S (TS)","TVA 3,8% sur achat B&S (TS)"
"","","","","","","","","","","","","tax","invoice","+400","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-400","ch_coa_1170","","",""
"vat_purchase_38_incl","3.8% INC","3.8% purchase on goods and services (incl.)","3.8% incl.","3.8","percent","purchase","tax_group_vat_38","True","","True","","base","invoice","","","VST 3,8% Mat.-/DL (inkl. MWST)","IVA 3,8% sull'acquisto di B&S (Incl. TS)","TVA 3,8% sur achat B&S (Incl. TS)"
"","","","","","","","","","","","","tax","invoice","+400","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-400","ch_coa_1170","","",""
"vat_purchase_38_invest","3.8% I OE","3.8% on invest. and others expenses","3.8% invest.","3.8","percent","purchase","tax_group_vat_38","","","True","","base","invoice","","","VST 3,8%  Inv./übr.BA (exkl. MWST)","IVA 3,8% su investimenti e altre voci (TS)","TVA 3,8% sur invest. et autres ch. (TS)"
"","","","","","","","","","","","","tax","invoice","+405","ch_coa_1171","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-405","ch_coa_1171","","",""
"vat_purchase_38_invest_incl","3.8% I OE INC","3.8% on invest. and others expenses (incl.)","3.8% invest. incl.","3.8","percent","purchase","tax_group_vat_38","True","","True","","base","invoice","","","VST 3,8%  Inv./übr.BA (inkl. MWST)","IVA 3,8% su investimenti e altre voci (incl. TS)","TVA 3,8% sur invest. et autres ch. (Incl. TS)"
"","","","","","","","","","","","","tax","invoice","+405","ch_coa_1171","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-405","ch_coa_1171","","",""
"vat_sale_81","8.1%","8.1% Sales","8.1%","8.1","percent","sale","tax_group_vat_81","","1","True","","base","invoice","+303a","","UST 8,1% Lief./DL (exkl. MWST)","IVA dovuta al 8,1% (TN)","TVA due à 8,1% (TN)"
"","","","","","","","","","","","","tax","invoice","+303b","ch_coa_2200","","",""
"","","","","","","","","","","","","base","refund","-303a","","","",""
"","","","","","","","","","","","","tax","refund","-303b","ch_coa_2200","","",""
"vat_sale_81_incl","8.1% INC","8.1% Sales (incl.)","8.1% incl.","8.1","percent","sale","tax_group_vat_81","True","1","True","","base","invoice","+303a","","UST 8,1% Lief./DL (inkl. MWST)","IVA dovuta al 8,1% (Incl. TN)","TVA due à 8,1% (Incl. TN)"
"","","","","","","","","","","","","tax","invoice","+303b","ch_coa_2200","","",""
"","","","","","","","","","","","","base","refund","-303a","","","",""
"","","","","","","","","","","","","tax","refund","-303b","ch_coa_2200","","",""
"vat_purchase_81","8.1%","8.1% on goods and services","8.1%","8.1","percent","purchase","tax_group_vat_81","","1","True","","base","invoice","","","VST 8,1% Mat.-/DL (exkl. MWST)","IVA 8,1% sull'acquisto di B&S (TN)","TVA 8,1% sur achat B&S (TN)"
"","","","","","","","","","","","","tax","invoice","+400","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-400","ch_coa_1170","","",""
"vat_purchase_81_incl","8.1% INC","8.1% on goods and services (incl.)","8.1% incl.","8.1","percent","purchase","tax_group_vat_81","True","1","True","","base","invoice","","","VST 8,1% Mat.-/DL (inkl. MWST)","IVA 8,1% sull'acquisto di B&S (Incl. TN)","TVA 8,1% sur achat B&S (Incl. TN)"
"","","","","","","","","","","","","tax","invoice","+400","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-400","ch_coa_1170","","",""
"vat_purchase_81_invest","8.1% I OE","8.1% on invest. and others expenses","8.1% invest.","8.1","percent","purchase","tax_group_vat_81","","","True","","base","invoice","","","VST 8,1% Inv./übr.BA (exkl. MWST)","IVA 8,1% su investimenti e altre voci (TN)","TVA 8,1% sur invest. et autres ch. (TN)"
"","","","","","","","","","","","","tax","invoice","+405","ch_coa_1171","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-405","ch_coa_1171","","",""
"vat_purchase_81_invest_incl","8.1% I OE INC","8.1% on invest. and others expenses (incl.)","8.1% invest. incl.","8.1","percent","purchase","tax_group_vat_81","True","","True","","base","invoice","","","VST 8,1% Inv./übr.BA (inkl. MWST)","IVA 8,1% su investimenti e altre voci (incl. TN)","TVA 8,1% sur invest. et autres ch. (Incl. TN)"
"","","","","","","","","","","","","tax","invoice","+405","ch_coa_1171","","",""
"","","","","","","","","","","","","base","refund","","","","",""
"","","","","","","","","","","","","tax","refund","-405","ch_coa_1171","","",""
"vat_purchase_81_return","8.1% R","8.1% Sales (return)","8.1% return","-8.1","percent","none","tax_group_vat_81","","0","True","","base","invoice","-383a","","UST 8,1% Bezugssteuer","IVA dovuta al 8,1% (TN) (rendimento)","TVA due à 8,1% (TN) (return)"
"","","","","","","","","","","","","tax","invoice","-383b","ch_coa_1170","","",""
"","","","","","","","","","","","","base","refund","+383a","","","",""
"","","","","","","","","","","","","tax","refund","+383b","ch_coa_1170","","",""
"vat_purchase_81_reverse","8.1% R C","8.1% on purchase of service abroad (reverse charge)","8.1% reverse","","group","purchase","tax_group_vat_81","","","True","vat_purchase_81_return,vat_purchase_81","","","","","BZS 8,1% Bezugssteuer","IVA 8,1% sull'acquisto di servizi all'estero (reverse charge)","TVA 8,1% sur achat service a l'étranger (reverse charge)"

```

## File: data\template\account.tax.group-ch.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id","name@de","name@fr","name@it","name@nl","name@zh_CN"
"tax_group_tva_0","VAT 0%","base.ch","ch_coa_2201","ch_coa_1176","MwSt. 0%","TVA 0%","IVA 0%","BTW 0%",""
"tax_group_tva_25","VAT 2.5%","base.ch","ch_coa_2201","ch_coa_1176","MwSt. 2,5%","TVA 2,5%","IVA 2,5%","",""
"tax_group_tva_37","VAT 3.7%","base.ch","ch_coa_2201","ch_coa_1176","MwSt. 3,7%","TVA 3,7%","IVA 3,7%","","TVA 3.7%"
"tax_group_tva_77","VAT 7.7%","base.ch","ch_coa_2201","ch_coa_1176","MwSt. 7,7%","TVA 7,7%","IVA 7,7%","","TVA 7.7%"
"tax_group_tva_100","VAT 100%","base.ch","ch_coa_2201","ch_coa_1176","MwSt. 100%","TVA 100%","IVA 100%","",""
"tax_group_vat_26","VAT 2.6%","base.ch","ch_coa_2201","ch_coa_1176","MwSt. 2,6%","TVA 2,6%","IVA 2,6%","",""
"tax_group_vat_38","VAT 3.8%","base.ch","ch_coa_2201","ch_coa_1176","MwSt. 3,8%","TVA 3,8%","IVA 3,8%","",""
"tax_group_vat_81","VAT 8.1%","base.ch","ch_coa_2201","ch_coa_1176","MwSt. 8,1%","TVA 8,1%","IVA 8,1%","",""

```

## File: migrations\0.0.0\pre-migrate-qr-template.py

```python
# -*- coding: utf-8 -*-


def migrate(cr, version):
    """ From 12.0, to saas-13.3, l10n_ch_swissqr_template
    used to inherit from another template. This isn't the case
    anymore since https://github.com/odoo/odoo/commit/719f087b1b5be5f1f276a0f87670830d073f6ef4
    (made in 12.0, and forward-ported). The module will not be updatable if we
    don't manually clean inherit_id.
    """
    cr.execute("""
        update ir_ui_view v
        set inherit_id = NULL, mode='primary'
        from ir_model_data mdata
        where
        v.id = mdata.res_id
        and mdata.model= 'ir.ui.view'
        and mdata.name = 'l10n_ch_swissqr_template'
        and mdata.module='l10n_ch';
    """)
```

## File: migrations\11.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'ch')], order="parent_path"):
        # We had corrupted data, handle the correction so the tax update can proceed.
        # See https://github.com/odoo/odoo/commit/7b07df873535446f97abc1de9176b9332de5cb07
        taxes_to_check = (f'{company.id}_vat_purchase_81_reverse', f'{company.id}_vat_77_purchase_reverse')
        tax_ids = env['ir.model.data'].search([
            ('name', 'in', taxes_to_check),
            ('model', '=', 'account.tax'),
        ]).mapped('res_id')
        for tax in env['account.tax'].browse(tax_ids).with_context(active_test=False):
            for child in tax.children_tax_ids:
                if child.type_tax_use not in ('none', tax.type_tax_use):
                    # set the child to it's parent's value
                    child.type_tax_use = tax.type_tax_use

        env['account.chart.template'].try_loading('ch', company)

```

## File: migrations\11.2\pre-migrate.py

```python
# -*- coding: utf-8 -*-


def migrate(cr, version):
    cr.execute("SELECT res_id FROM ir_model_data WHERE module = 'l10n_ch' AND name='account_tax_report_line_chtax_solde_formula'")

    expression_id = cr.fetchone()

    if expression_id:
        cr.execute(
            "DELETE FROM account_report_external_value WHERE target_report_expression_id = %s",
            [expression_id[0]]
        )

```

## File: migrations\11.3\end-migrate.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env["res.company"].search([("chart_template", "=", "ch")], order="parent_path"):
        env["account.chart.template"].try_loading("ch", company)

```

## File: migrations\9.0.9.0\pre-set_tags_and_taxes_updatable.py

```python
# -*- coding: utf-8 -*-

import odoo

def migrate(cr, version):
    registry = odoo.registry(cr.dbname)
    from odoo.addons.account.models.chart_template import migrate_set_tags_and_taxes_updatable
    migrate_set_tags_and_taxes_updatable(cr, registry, 'l10n_ch')


```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import re

from odoo import models, fields, api, _
from odoo.exceptions import UserError
from odoo.tools.misc import mod10r

L10N_CH_QRR_NUMBER_LENGTH = 27

class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_ch_is_qr_valid = fields.Boolean(compute='_compute_l10n_ch_qr_is_valid', help="Determines whether an invoice can be printed as a QR or not")

    @api.depends('partner_id', 'currency_id', 'display_qr_code')
    def _compute_l10n_ch_qr_is_valid(self):
        for move in self:
            error_messages = move.partner_bank_id._get_error_messages_for_qr('ch_qr', move.partner_id, move.currency_id)
            move.l10n_ch_is_qr_valid = (
                move.display_qr_code and
                move.move_type == 'out_invoice' and
                not error_messages and
                (move.company_id.account_fiscal_country_id.code == 'CH' or move._is_swiss_qr_iban())
            )

    @api.depends('company_id', 'state')
    def _compute_display_qr_code(self):
        # Extends account
        super()._compute_display_qr_code()
        moves_ch = self.filtered(lambda m: m._is_swiss_qr_iban())
        for move in moves_ch:
            move.display_qr_code = move.state != 'draft'

    def get_l10n_ch_qrr_number(self):
        """Generates the QRR reference.
        QRR references are 27 characters long.

        The invoice sequence number is used, removing each of its non-digit characters,
        and pad the unused spaces on the left of this number with zeros.
        The last digit is a checksum (mod10r).
        """
        self.ensure_one()
        if self.partner_bank_id.l10n_ch_qr_iban and self.l10n_ch_is_qr_valid and self.name:
            invoice_ref = re.sub(r'[^\d]', '', self.name)
            return self._compute_qrr_number(invoice_ref)
        else:
            return False

    @api.model
    def _compute_qrr_number(self, invoice_ref):
        # keep only the last digits if it exceed boundaries
        ref_payload_len = L10N_CH_QRR_NUMBER_LENGTH - 1
        extra = len(invoice_ref) - ref_payload_len
        if extra > 0:
            invoice_ref = invoice_ref[extra:]
        internal_ref = invoice_ref.zfill(ref_payload_len)
        return mod10r(internal_ref)

    def _get_invoice_reference_ch_invoice(self):
        """ This sets QRR reference number which is generated based on customer's `Bank Account` and set it as
        `Payment Reference` of the invoice when invoice's journal is using Switzerland's communication standard
        """
        self.ensure_one()
        return self.get_l10n_ch_qrr_number()

    def _get_invoice_reference_ch_partner(self):
        """ This sets QRR reference number which is generated based on customer's `Bank Account` and set it as
        `Payment Reference` of the invoice when invoice's journal is using Switzerland's communication standard
        """
        self.ensure_one()
        return self.get_l10n_ch_qrr_number()

    def _is_swiss_qr_iban(self):
        """QR code is also printed if the fiscal country is not Switzerland but the receivable account is eligible"""
        self.ensure_one()
        iban = (self.partner_bank_id.acc_number or '').replace(' ', '')
        return self.partner_bank_id.acc_type == 'iban' and iban.startswith('CH') and iban[4:9].isdigit() and 30000 <= int(iban[4:9]) <= 31999

    @api.model
    def space_qrr_reference(self, qrr_ref):
        """ Makes the provided QRR reference human-friendly, spacing its elements
        by blocks of 5 from right to left.
        """
        spaced_qrr_ref = ''
        i = len(qrr_ref) # i is the index after the last index to consider in substrings
        while i > 0:
            spaced_qrr_ref = qrr_ref[max(i-5, 0) : i] + ' ' + spaced_qrr_ref
            i -= 5
        return spaced_qrr_ref

    @api.model
    def space_scor_reference(self, iso11649_ref):
        """ Makes the provided SCOR reference human-friendly, spacing its elements
        by blocks of 5 from right to left.
        """
        return ' '.join(iso11649_ref[i:i + 4] for i in range(0, len(iso11649_ref), 4))

    def l10n_ch_action_print_qr(self):
        '''
        Checks that all invoices can be printed in the QR format.
        If so, launches the printing action.
        Else, triggers the l10n_ch wizard that will display the informations.
        '''
        if any(x.move_type != 'out_invoice' for x in self):
            raise UserError(_("Only customers invoices can be QR-printed."))
        if False in self.mapped('l10n_ch_is_qr_valid'):
            return {
                'name': (_("Some invoices could not be printed in the QR format")),
                'type': 'ir.actions.act_window',
                'res_model': 'l10n_ch.qr_invoice.wizard',
                'view_mode': 'form',
                'target': 'new',
                'context': {'active_ids': self.ids},
            }
        return self.env.ref('account.account_invoices').report_action(self)

    def _l10n_ch_dispatch_invoices_to_print(self):
        qr_invs = self.filtered('l10n_ch_is_qr_valid')
        return {
            'qr': qr_invs,
            'classic': self - qr_invs,
        }

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api

from odoo.exceptions import ValidationError

from odoo.addons.base_iban.models.res_partner_bank import validate_iban
from odoo.addons.base.models.res_bank import sanitize_account_number


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    invoice_reference_model = fields.Selection(selection_add=[
        ('ch', 'Switzerland')
    ], ondelete={'ch': lambda recs: recs.write({'invoice_reference_model': 'odoo'})})

    def _process_reference_for_sale_order(self, order_reference):
        '''
        Returns the order reference to be used for the payment, respecting the QRR standard.
        '''
        self.ensure_one()
        if self.invoice_reference_model == 'ch':
            # converting the sale order name into a unique number. Letters are converted to their base10 value
            invoice_ref = "".join([a if a.isdigit() else str(ord(a)) for a in order_reference])
            # id_number = self.company_id.bank_ids.l10n_ch_postal or ''
            order_reference = self.env['account.move']._compute_qrr_number(invoice_ref)
            return order_reference
        return super()._process_reference_for_sale_order(order_reference)

```

## File: models\account_payment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import re

from odoo import _, models, fields, api
from odoo.tools import mod10r


class AccountPayment(models.Model):
    _inherit = "account.payment"

    l10n_ch_reference_warning_msg = fields.Char(compute='_compute_l10n_ch_reference_warning_msg')

    @api.onchange('partner_id', 'ref', 'payment_type')
    def _compute_l10n_ch_reference_warning_msg(self):
        for payment in self:
            if payment.payment_type == 'outbound' and\
                    payment.partner_id.country_code in ['CH', 'LI'] and\
                    payment.partner_bank_id.l10n_ch_qr_iban and\
                    not payment._l10n_ch_reference_is_valid(payment.ref):
                payment.l10n_ch_reference_warning_msg = _("Please fill in a correct QRR reference in the payment reference. The banks will refuse your payment file otherwise.")
            else:
                payment.l10n_ch_reference_warning_msg = False

    def _l10n_ch_reference_is_valid(self, payment_reference):
        """Check if this invoice has a valid reference (for Switzerland)
        e.g.
        000000000000000000000012371
        210000000003139471430009017
        21 00000 00003 13947 14300 09017
        """
        self.ensure_one()
        if not payment_reference:
            return False
        ref = payment_reference.replace(' ', '')
        if re.match(r'^(\d{2,27})$', ref):
            return ref == mod10r(ref[:-1])
        return False

```

## File: models\ir_actions_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
import io
from odoo import api, models
from odoo.tools.pdf import OdooPdfFileReader, OdooPdfFileWriter
from pathlib import Path
from reportlab.graphics.shapes import Drawing as ReportLabDrawing, Image as ReportLabImage
from reportlab.lib.units import mm

CH_QR_CROSS_SIZE_RATIO = 0.1522 # Ratio between the side length of the Swiss QR-code cross image and the QR-code's
CH_QR_CROSS_FILE = Path('../static/src/img/CH-Cross_7mm.png') # Image file containing the Swiss QR-code cross to add on top of the QR-code

class IrActionsReport(models.Model):
    _inherit = 'ir.actions.report'

    @api.model
    def get_available_barcode_masks(self):
        rslt = super(IrActionsReport, self).get_available_barcode_masks()
        rslt['ch_cross'] = self.apply_qr_code_ch_cross_mask
        return rslt

    @api.model
    def apply_qr_code_ch_cross_mask(self, width, height, barcode_drawing):
        assert isinstance(barcode_drawing, ReportLabDrawing)
        zoom_x = barcode_drawing.transform[0]
        zoom_y = barcode_drawing.transform[3]
        cross_width = CH_QR_CROSS_SIZE_RATIO * width
        cross_height = CH_QR_CROSS_SIZE_RATIO * height
        cross_path = Path(__file__).absolute().parent / CH_QR_CROSS_FILE
        qr_cross = ReportLabImage((width/2 - cross_width/2) / zoom_x, (height/2 - cross_height/2) / zoom_y, cross_width / zoom_x, cross_height / zoom_y, cross_path.as_posix())
        barcode_drawing.add(qr_cross)

    def _render_qweb_pdf_prepare_streams(self, report_ref, data, res_ids=None):
        # OVERRIDE
        res = super()._render_qweb_pdf_prepare_streams(report_ref, data, res_ids)
        if not res_ids:
            return res
        report = self._get_report(report_ref)
        if self._is_invoice_report(report_ref):
            invoices = self.env[report.model].browse(res_ids)
            # Determine which invoices need a QR.
            qr_inv_ids = []
            for invoice in invoices:
                # avoid duplicating existing streams
                if report.attachment_use and report.retrieve_attachment(invoice):
                    continue
                if invoice.l10n_ch_is_qr_valid:
                    qr_inv_ids.append(invoice.id)
            # Render the additional reports.
            streams_to_append = {}
            if qr_inv_ids:
                qr_res = self._render_qweb_pdf_prepare_streams(
                    'l10n_ch.l10n_ch_qr_report',
                    {
                        **data,
                        'skip_headers': False,
                    },
                    res_ids=qr_inv_ids,
                )
                header = self.env.ref('l10n_ch.l10n_ch_qr_header', raise_if_not_found=False)
                if header:
                    # Make a separated rendering to get the a page containing the company header. Then, merge the qr bill with it.

                    header_res = self._render_qweb_pdf_prepare_streams(
                        'l10n_ch.l10n_ch_qr_header',
                        {
                            **data,
                            'skip_headers': True,
                        },
                        res_ids=qr_inv_ids,
                    )

                    for invoice_id, stream in qr_res.items():
                        qr_pdf = OdooPdfFileReader(stream['stream'], strict=False)
                        header_pdf = OdooPdfFileReader(header_res[invoice_id]['stream'], strict=False)

                        page = header_pdf.getPage(0)
                        page.mergePage(qr_pdf.getPage(0))

                        output_pdf = OdooPdfFileWriter()
                        output_pdf.addPage(page)
                        new_pdf_stream = io.BytesIO()
                        output_pdf.write(new_pdf_stream)
                        streams_to_append[invoice_id] = {'stream': new_pdf_stream}
                else:
                    for invoice_id, stream in qr_res.items():
                        streams_to_append[invoice_id] = stream

            # Add to results
            for invoice_id, additional_stream in streams_to_append.items():
                invoice_stream = res[invoice_id]['stream']
                writer = OdooPdfFileWriter()
                writer.appendPagesFromReader(OdooPdfFileReader(invoice_stream, strict=False))
                writer.appendPagesFromReader(OdooPdfFileReader(additional_stream['stream'], strict=False))
                new_pdf_stream = io.BytesIO()
                writer.write(new_pdf_stream)
                res[invoice_id]['stream'] = new_pdf_stream
                invoice_stream.close()
                additional_stream['stream'].close()
        return res

    def get_paperformat(self):
        if self.env.context.get('snailmail_layout'):
            if self.report_name == 'l10n_ch.qr_report_main':
                return self.env.ref('l10n_ch.paperformat_euro_no_margin')
            if self.report_name == 'l10n_ch.qr_report_header':
                return self.env.ref('l10n_din5008.paperformat_euro_din')
        return super(IrActionsReport, self).get_paperformat()

```

## File: models\res_bank.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from stdnum.util import clean

from odoo import api, fields, models, _
from odoo.addons.base.models.res_bank import sanitize_account_number
from odoo.addons.base_iban.models.res_partner_bank import normalize_iban, pretty_iban, validate_iban
from odoo.exceptions import ValidationError, UserError
from odoo.tools.misc import mod10r


def validate_qr_iban(qr_iban):
    # Check first if it's a valid IBAN.
    validate_iban(qr_iban)

    # We sanitize first so that _check_qr_iban_range() can extract correct IID from IBAN to validate it.
    sanitized_qr_iban = sanitize_account_number(qr_iban)

    if sanitized_qr_iban[:2] not in ['CH', 'LI']:
        raise ValidationError(_("QR-IBAN numbers are only available in Switzerland."))

    # Now, check if it's valid QR-IBAN (based on its IID).
    if not check_qr_iban_range(sanitized_qr_iban):
        raise ValidationError(_("QR-IBAN %r is invalid.", qr_iban))

    return True

def check_qr_iban_range(iban):
    if not iban or len(iban) < 9:
        return False
    iid_start_index = 4
    iid_end_index = 8
    iid = iban[iid_start_index : iid_end_index+1]
    return re.match(r'\d+', iid) and 30000 <= int(iid) <= 31999 # Those values for iid are reserved for QR-IBANs only


class ResPartnerBank(models.Model):
    _inherit = 'res.partner.bank'

    l10n_ch_qr_iban = fields.Char(string='QR-IBAN',
                                  compute='_compute_l10n_ch_qr_iban',
                                  store=True,
                                  readonly=False,
                                  help="Put the QR-IBAN here for your own bank accounts.  That way, you can "
                                       "still use the main IBAN in the Account Number while you will see the "
                                       "QR-IBAN for the barcode.  ")

    # fields to configure payment slip generation
    l10n_ch_display_qr_bank_options = fields.Boolean(compute='_compute_l10n_ch_display_qr_bank_options')

    @api.depends('partner_id', 'company_id')
    def _compute_l10n_ch_display_qr_bank_options(self):
        for bank in self:
            if bank.partner_id:
                bank.l10n_ch_display_qr_bank_options = bank.partner_id.ref_company_ids.country_id.code in ('CH', 'LI')
            elif bank.company_id:
                bank.l10n_ch_display_qr_bank_options = bank.company_id.account_fiscal_country_id.code in ('CH', 'LI')
            else:
                bank.l10n_ch_display_qr_bank_options = self.env.company.account_fiscal_country_id.code in ('CH', 'LI')

    @api.depends('acc_number')
    def _compute_l10n_ch_qr_iban(self):
        for record in self:
            try:
                validate_qr_iban(record.acc_number)
                valid_qr_iban = True
            except ValidationError:
                valid_qr_iban = False
            if valid_qr_iban:
                record.l10n_ch_qr_iban = record.sanitized_acc_number
            else:
                record.l10n_ch_qr_iban = None

    @api.model_create_multi
    def create(self, vals_list):
        for vals in vals_list:
            if vals.get('l10n_ch_qr_iban'):
                validate_qr_iban(vals['l10n_ch_qr_iban'])
                vals['l10n_ch_qr_iban'] = pretty_iban(normalize_iban(vals['l10n_ch_qr_iban']))
        return super().create(vals_list)

    def write(self, vals):
        if vals.get('l10n_ch_qr_iban'):
            validate_qr_iban(vals['l10n_ch_qr_iban'])
            vals['l10n_ch_qr_iban'] = pretty_iban(normalize_iban(vals['l10n_ch_qr_iban']))
        return super().write(vals)

    def _l10n_ch_get_qr_vals(self, amount, currency, debtor_partner, free_communication, structured_communication):
        comment = ""
        if free_communication:
            comment = (free_communication[:137] + '...') if len(free_communication) > 140 else free_communication

        creditor_addr_1, creditor_addr_2 = self._get_partner_address_lines(self.partner_id)
        debtor_addr_1, debtor_addr_2 = self._get_partner_address_lines(debtor_partner)

        # Compute reference type (empty by default, only mandatory for QR-IBAN,
        # and must then be 27 characters-long, with mod10r check digit as the 27th one)
        reference_type = 'NON'
        reference = ''
        acc_number = self.sanitized_acc_number

        if self.l10n_ch_qr_iban:
            # _check_for_qr_code_errors ensures we can't have a QR-IBAN without a QR-reference here
            reference_type = 'QRR'
            reference = structured_communication
            acc_number = sanitize_account_number(self.l10n_ch_qr_iban)
        elif self._is_iso11649_reference(structured_communication):
            reference_type = 'SCOR'
            reference = structured_communication.replace(' ', '')

        currency = currency or self.currency_id or self.company_id.currency_id

        return [
            'SPC',                                                # QR Type
            '0200',                                               # Version
            '1',                                                  # Coding Type
            acc_number,                                           # IBAN / QR-IBAN
            'K',                                                  # Creditor Address Type
            (self.acc_holder_name or self.partner_id.name)[:70],  # Creditor Name
            creditor_addr_1,                                      # Creditor Address Line 1
            creditor_addr_2,                                      # Creditor Address Line 2
            '',                                                   # Creditor Postal Code (empty, since we're using combined addres elements)
            '',                                                   # Creditor Town (empty, since we're using combined addres elements)
            self.partner_id.country_id.code,                      # Creditor Country
            '',                                                   # Ultimate Creditor Address Type
            '',                                                   # Name
            '',                                                   # Ultimate Creditor Address Line 1
            '',                                                   # Ultimate Creditor Address Line 2
            '',                                                   # Ultimate Creditor Postal Code
            '',                                                   # Ultimate Creditor Town
            '',                                                   # Ultimate Creditor Country
            '{:.2f}'.format(amount),                              # Amount
            currency.name,                                        # Currency
            'K',                                                  # Ultimate Debtor Address Type
            debtor_partner.commercial_partner_id.name[:70],       # Ultimate Debtor Name
            debtor_addr_1,                                        # Ultimate Debtor Address Line 1
            debtor_addr_2,                                        # Ultimate Debtor Address Line 2
            '',                                                   # Ultimate Debtor Postal Code (not to be provided for address type K)
            '',                                                   # Ultimate Debtor Postal City (not to be provided for address type K)
            debtor_partner.country_id.code,                       # Ultimate Debtor Postal Country
            reference_type,                                       # Reference Type
            reference,                                            # Reference
            comment,                                              # Unstructured Message
            'EPD',                                                # Mandatory trailer part
        ]

    def _get_qr_vals(self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication):
        if qr_method == 'ch_qr':
            return self._l10n_ch_get_qr_vals(amount, currency, debtor_partner, free_communication, structured_communication)
        return super()._get_qr_vals(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)

    def _get_qr_code_generation_params(self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication):
        if qr_method == 'ch_qr':
            return {
                'barcode_type': 'QR',
                'width': 256,
                'height': 256,
                'quiet': 1,
                'mask': 'ch_cross',
                'value': '\n'.join(self._get_qr_vals(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)),
                # Swiss QR code requires Error Correction Level = 'M' by specification
                'barLevel': 'M',
            }
        return super()._get_qr_code_generation_params(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)

    def _get_partner_address_lines(self, partner):
        """ Returns a tuple of two elements containing the address lines to use
        for this partner. Line 1 contains the street and number, line 2 contains
        zip and city. Those two lines are limited to 70 characters
        """
        streets = [partner.street, partner.street2]
        line_1 = ' '.join(filter(None, streets))
        line_2 = partner.zip + ' ' + partner.city
        return line_1[:70], line_2[:70]

    @api.model
    def _is_qr_reference(self, reference):
        """ Checks whether the given reference is a QR-reference, i.e. it is
        made of 27 digits, the 27th being a mod10r check on the 26 previous ones.
        """
        return reference \
            and len(reference) == 27 \
            and re.match(r'\d+$', reference) \
            and reference == mod10r(reference[:-1])

    @api.model
    def _is_iso11649_reference(self, reference):
        """ Checks whether the given reference is a ISO11649 (SCOR) reference.
        """
        return reference \
               and len(reference) >= 5 \
               and len(reference) <= 25 \
               and reference.startswith('RF') \
               and int(''.join(str(int(x, 36)) for x in clean(reference[4:] + reference[:4], ' -.,/:').upper().strip())) % 97 == 1
               # see https://github.com/arthurdejong/python-stdnum/blob/master/stdnum/iso11649.py

    def _get_error_messages_for_qr(self, qr_method, debtor_partner, currency):
        def _get_error_for_ch_qr():
            error_messages = [_("The Swiss QR code could not be generated for the following reason(s):")]
            if self.acc_type != 'iban':
                error_messages.append(_("The account type isn't QR-IBAN or IBAN."))
            if not debtor_partner or debtor_partner.country_id.code not in ('CH', 'LI'):
                error_messages.append(_("The debtor partner's address isn't located in Switzerland."))
            if currency.id not in (self.env.ref('base.EUR').id, self.env.ref('base.CHF').id):
                error_messages.append(_("The currency isn't EUR nor CHF."))
            return '\r\n'.join(error_messages) if len(error_messages) > 1 else None

        if qr_method == 'ch_qr':
            return _get_error_for_ch_qr()
        return super()._get_error_messages_for_qr(qr_method, debtor_partner, currency)

    def _check_for_qr_code_errors(self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication):
        def _partner_fields_set(partner):
            return partner.zip and \
                   partner.city and \
                   partner.country_id.code and \
                   (partner.street or partner.street2)

        if qr_method == 'ch_qr':
            if not _partner_fields_set(self.partner_id):
                return _("The partner set on the bank account meant to receive the payment (%s) must have a complete postal address (street, zip, city and country).", self.acc_number)

            if debtor_partner and not _partner_fields_set(debtor_partner):
                return _("The partner must have a complete postal address (street, zip, city and country).")

            if self.l10n_ch_qr_iban and not self._is_qr_reference(structured_communication):
                return _("When using a QR-IBAN as the destination account of a QR-code, the payment reference must be a QR-reference.")

        return super()._check_for_qr_code_errors(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)

    @api.model
    def _get_available_qr_methods(self):
        rslt = super()._get_available_qr_methods()
        rslt.append(('ch_qr', _("Swiss QR bill"), 10))
        return rslt

```

## File: models\template_ch.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ch')
    def _get_ch_template_data(self):
        return {
            'code_digits': '4',
            'property_account_receivable_id': 'ch_coa_1100',
            'property_account_payable_id': 'ch_coa_2000',
            'property_account_expense_categ_id': 'ch_coa_4200',
            'property_account_income_categ_id': 'ch_coa_3200',
        }

    @template('ch', 'res.company')
    def _get_ch_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.ch',
                'bank_account_code_prefix': '102',
                'cash_account_code_prefix': '100',
                'transfer_account_code_prefix': '1090',
                'account_default_pos_receivable_account_id': 'ch_coa_1101',
                'income_currency_exchange_account_id': 'ch_coa_3806',
                'expense_currency_exchange_account_id': 'ch_coa_4906',
                'account_journal_early_pay_discount_loss_account_id': 'ch_coa_4901',
                'account_journal_early_pay_discount_gain_account_id': 'ch_coa_3801',
                'default_cash_difference_expense_account_id': 'ch_coa_4991',
                'default_cash_difference_income_account_id': 'ch_coa_4992',
                'account_sale_tax_id': 'vat_sale_81',
                'account_purchase_tax_id': 'vat_purchase_81',
                'external_report_layout_id': 'l10n_din5008.external_layout_din5008',
                'paperformat_id': 'l10n_din5008.paperformat_euro_din',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_ch
from . import account_invoice
from . import account_journal
from . import res_bank
from . import ir_actions_report
from . import account_payment

```

## File: report\swissqr_report.py

```python
# -*- coding:utf-8 -*-
from odoo import api, models

class ReportSwissQR(models.AbstractModel):
    _name = 'report.l10n_ch.qr_report_main'
    _description = 'Swiss QR-bill report'

    @api.model
    def _get_report_values(self, docids, data=None):
        docs = self.env['account.move'].browse(docids)

        qr_code_urls = {}
        for invoice in docs:
            qr_code_urls[invoice.id] = invoice.partner_bank_id.build_qr_code_base64(invoice.amount_residual, invoice.ref or invoice.name, invoice.payment_reference, invoice.currency_id, invoice.partner_id, qr_method='ch_qr', silent_errors=False)

        return {
            'doc_ids': docids,
            'doc_model': 'account.move',
            'docs': docs,
            'qr_code_urls': qr_code_urls,
        }

```

## File: report\swissqr_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="paperformat_euro_no_margin" model="report.paperformat">
            <field name="name">European A4 without borders</field>
            <field name="default" eval="False" />
            <field name="format">A4</field>
            <field name="orientation">Portrait</field>
            <field name="margin_top">0</field>
            <field name="margin_bottom">0</field>
            <field name="margin_left">0</field>
            <field name="margin_right">0</field>
            <field name="header_line" eval="False" />
            <field name="header_spacing">0</field>
        </record>

        <record id="l10n_ch_qr_report" model="ir.actions.report">
            <field name="name">QR-bill</field>
            <field name="model">account.move</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">l10n_ch.qr_report_main</field>
            <field name="report_file">l10n_ch.qr_report_main</field>
            <field name="print_report_name">'QR-bill-%s' % object.name</field>
            <field name="paperformat_id" ref="l10n_ch.paperformat_euro_no_margin"/>
        </record>

        <record id="l10n_ch_qr_header" model="ir.actions.report">
            <field name="name">QR-bill Header</field>
            <field name="model">account.move</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">l10n_ch.qr_report_header</field>
            <field name="report_file">l10n_ch.qr_report_header</field>
        </record>

        <template id="l10n_ch_header_template">
            <t t-call="web.external_layout">
                <!--The following elements are necessary for the header to be displayed correctly.-->
                <br/>
                <p>&amp;nbsp;</p>
            </t>
        </template>

        <template id="l10n_ch_swissqr_template">
            <div class="article" t-att-data-oe-model="o._name" t-att-data-oe-id="o.id">
                <t t-set="o" t-value="o.with_context(lang=lang)"/>
                <t t-set="company" t-value="o.company_id"/>
                <t t-set="formated_amount" t-value="'{:,.2f}'.format(o.amount_residual).replace(',','\xa0')"/>

                <t t-set="is_qrr" t-value="o.partner_bank_id.l10n_ch_qr_iban"/>
                <t t-set="is_scor" t-value="o.partner_bank_id._is_iso11649_reference(o.payment_reference)"/>
                <div class="swissqr_content_v2">

                    <div class="swissqr_receipt">

                        <img src="/l10n_ch/static/src/img/scissors_h.png" class="scissors horizontal_scissors"/>

                        <div id="receipt_title_zone" class="swissqr_section_title">
                            <span>Receipt</span>
                        </div>

                        <div id="receipt_indication_zone" class="receipt_indication_zone">

                            <div class="swissqr_text title">
                                <span>Account / Payable to</span>
                            </div>
                            <div class="swissqr_text content">
                                <span t-field="o.partner_bank_id.acc_number" t-if="not o.partner_bank_id.l10n_ch_qr_iban"/>
                                <span t-field="o.partner_bank_id.l10n_ch_qr_iban" t-if="o.partner_bank_id.l10n_ch_qr_iban"/>
                                <br/>
                                <span t-out="o.partner_bank_id.partner_id.name or o.company_id.name"/><br/>
                                <span t-field="o.company_id.street"/><br/>
                                <span t-field="o.company_id.country_id.code"/>
                                <span t-field="o.company_id.zip"/>
                                <span t-field="o.company_id.city"/><br/>
                                <br/>
                            </div>

                            <t t-if="is_qrr or is_scor">
                                <div class="swissqr_text title">
                                    <span>Reference</span>
                                </div>
                            </t>
                            <t t-if="is_qrr">
                                <div class="swissqr_text content">
                                    <span t-out="o.space_qrr_reference(o.payment_reference)"/><br/>
                                    <br/>
                                </div>
                            </t>
                            <t t-if="is_scor">
                                <div class="swissqr_text content">
                                    <span t-out="o.space_scor_reference(o.payment_reference)"/><br/>
                                    <br/>
                                </div>
                            </t>

                            <div class="swissqr_text title">
                                <span>Payable by</span>
                            </div>
                            <div class="swissqr_text content">
                                <span t-field="o.partner_id.commercial_partner_id.name"/><br/>
                                <span t-field="o.partner_id.street"/>
                                <span t-field="o.partner_id.street2"/><br/>
                                <span t-field="o.partner_id.country_id.code"/>
                                <span t-field="o.partner_id.zip"/>
                                <span t-field="o.partner_id.city"/>
                            </div>

                        </div>

                        <div id="receipt_amount_zone" class="swissqr_column_left receipt_amount_zone">
                            <div class="swissqr_text">
                                <div class="column">
                                    <div class="title">
                                        <span>Currency</span>
                                    </div>
                                    <div class="content">
                                        <span t-field="o.currency_id.name"/>
                                    </div>
                                </div>
                                <div class="column">
                                    <div class="title">
                                        <span>Amount</span>
                                    </div>
                                    <div class="content">
                                        <span t-out="formated_amount"/>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div id="receipt_acceptance_point_zone" class="receipt_acceptance_point_zone">
                            <div class="swissqr_text content">
                                <span class="title">Acceptance point</span>
                            </div>
                        </div>

                    </div>

                    <div class="swissqr_body">

                        <img src="/l10n_ch/static/src/img/scissors_v.png" class="scissors vertical_scissors"/>

                        <div class="swissqr_column_left">

                            <div class="swissqr_section_title">
                                <span>Payment part</span>
                            </div>

                            <img class="swissqr" t-att-src="qr_code_urls[o.id]"/>

                            <div id="amount_zone" class="amount_zone">
                                <div class="swissqr_text">
                                    <div class="column">
                                        <div class="title">
                                            <span>Currency</span>
                                        </div>
                                        <div class="content">
                                            <span t-field="o.currency_id.name"/>
                                        </div>
                                    </div>
                                    <div class="column">
                                        <div class="title">
                                            <span>Amount</span><br/>
                                        </div>
                                        <div class="content">
                                            <span t-out="formated_amount"/>
                                        </div>
                                    </div>
                                </div>
                            </div>

                        </div>

                        <div id="indications_zone" class="swissqr_column_right">
                            <div class="swissqr_text title">
                                <span>Account / Payable to</span><br/>
                            </div>
                            <div class="swissqr_text content">
                                <span t-field="o.partner_bank_id.acc_number" t-if="not o.partner_bank_id.l10n_ch_qr_iban"/>
                                <span t-field="o.partner_bank_id.l10n_ch_qr_iban" t-if="o.partner_bank_id.l10n_ch_qr_iban"/>
                                <br/>
                                <span t-out="o.partner_bank_id.partner_id.name or o.company_id.name"/><br/>
                                <span t-field="o.company_id.street"/><br/>
                                <span t-field="o.company_id.country_id.code"/>
                                <span t-field="o.company_id.zip"/>
                                <span t-field="o.company_id.city"/><br/>
                                <br/>
                            </div>

                            <t t-if="is_qrr or is_scor">
                                <div class="swissqr_text title">
                                    <span class="title">Reference</span>
                                </div>
                            </t>
                            <t t-if="is_qrr">
                                <div class="swissqr_text content">
                                    <span t-out="o.space_qrr_reference(o.payment_reference)"/><br/>
                                    <br/>
                                </div>
                            </t>
                            <t t-if="is_scor">
                                <div class="swissqr_text content">
                                    <span t-out="o.space_scor_reference(o.payment_reference)"/><br/>
                                    <br/>
                                </div>
                            </t>

                            <t t-set="additional_info" t-value="(o.ref or o.name if is_qrr or is_scor else o.payment_reference or o.ref or o.name)"/>
                            <t t-if="additional_info">
                                <div class="swissqr_text title">
                                    <span>Additional information</span>
                                </div>
                                <div class="swissqr_text content">
                                    <span t-out="additional_info"/><br/>
                                    <br/>
                                </div>
                            </t>

                            <div class="swissqr_text title">
                                <span>Payable by</span>
                            </div>
                            <div class="swissqr_text content">
                                <span t-field="o.partner_id.commercial_partner_id.name"/><br/>
                                <span t-field="o.partner_id.street"> </span>
                                <span t-field="o.partner_id.street2"/><br/>
                                <span t-field="o.partner_id.country_id.code"/>
                                <span t-field="o.partner_id.zip"/>
                                <span t-field="o.partner_id.city"/><br/>
                                <br/>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </template>

        <template id="l10n_ch.qr_report_main">
            <t t-call="web.html_container">
                <t t-foreach="docs" t-as="o">
                    <t t-set="lang" t-value="o.partner_id.lang"/>
                    <t t-call="l10n_ch.l10n_ch_swissqr_template" t-lang="lang"/>
                </t>
            </t>
        </template>

        <template id="l10n_ch.qr_report_header">
            <t t-call="web.html_container">
                <t t-foreach="docs" t-as="invoice">
                    <t t-set="o" t-value="invoice"/>
                    <t t-set="lang" t-value="o.partner_id.lang"/>
                    <t t-call="l10n_ch.l10n_ch_header_template" t-lang="lang"/>
                </t>
            </t>
        </template>

        <template id="minimal_layout_with_report_attribute" inherit_id="web.minimal_layout">
            <body position="attributes">
                <attribute name="t-att-data-report-id">report_xml_id</attribute>
            </body>
        </template>
    </data>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding:utf-8 -*-

from . import swissqr_report
```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_ch_qr_invoice_wizard,l10n_ch.qr_invoice.wizard,model_l10n_ch_qr_invoice_wizard,account.group_account_invoice,1,1,1,0

```

## File: static\src\font\liberation_sans_license.txt

```text
Digitized data copyright (c) 2010 Google Corporation
	with Reserved Font Arimo, Tinos and Cousine.
Copyright (c) 2012 Red Hat, Inc.
	with Reserved Font Name Liberation.

This Font Software is licensed under the SIL Open Font License,
Version 1.1.

This license is copied below, and is also available with a FAQ at:
http://scripts.sil.org/OFL

SIL OPEN FONT LICENSE Version 1.1 - 26 February 2007

PREAMBLE The goals of the Open Font License (OFL) are to stimulate
worldwide development of collaborative font projects, to support the font
creation efforts of academic and linguistic communities, and to provide
a free and open framework in which fonts may be shared and improved in
partnership with others.

The OFL allows the licensed fonts to be used, studied, modified and
redistributed freely as long as they are not sold by themselves.
The fonts, including any derivative works, can be bundled, embedded,
redistributed and/or sold with any software provided that any reserved
names are not used by derivative works.  The fonts and derivatives,
however, cannot be released under any other type of license.  The
requirement for fonts to remain under this license does not apply to
any document created using the fonts or their derivatives.

 

DEFINITIONS
"Font Software" refers to the set of files released by the Copyright
Holder(s) under this license and clearly marked as such.
This may include source files, build scripts and documentation.

"Reserved Font Name" refers to any names specified as such after the
copyright statement(s).

"Original Version" refers to the collection of Font Software components
as distributed by the Copyright Holder(s).

"Modified Version" refers to any derivative made by adding to, deleting,
or substituting ? in part or in whole ?
any of the components of the Original Version, by changing formats or
by porting the Font Software to a new environment.

"Author" refers to any designer, engineer, programmer, technical writer
or other person who contributed to the Font Software.


PERMISSION & CONDITIONS

Permission is hereby granted, free of charge, to any person obtaining a
copy of the Font Software, to use, study, copy, merge, embed, modify,
redistribute, and sell modified and unmodified copies of the Font
Software, subject to the following conditions:

1) Neither the Font Software nor any of its individual components,in
   Original or Modified Versions, may be sold by itself.

2) Original or Modified Versions of the Font Software may be bundled,
   redistributed and/or sold with any software, provided that each copy
   contains the above copyright notice and this license. These can be
   included either as stand-alone text files, human-readable headers or
   in the appropriate machine-readable metadata fields within text or
   binary files as long as those fields can be easily viewed by the user.

3) No Modified Version of the Font Software may use the Reserved Font
   Name(s) unless explicit written permission is granted by the
   corresponding Copyright Holder. This restriction only applies to the
   primary font name as presented to the users.

4) The name(s) of the Copyright Holder(s) or the Author(s) of the Font
   Software shall not be used to promote, endorse or advertise any
   Modified Version, except to acknowledge the contribution(s) of the
   Copyright Holder(s) and the Author(s) or with their explicit written
   permission.

5) The Font Software, modified or unmodified, in part or in whole, must
   be distributed entirely under this license, and must not be distributed
   under any other license. The requirement for fonts to remain under
   this license does not apply to any document created using the Font
   Software.


 
TERMINATION
This license becomes null and void if any of the above conditions are not met.

 

DISCLAIMER
THE FONT SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO ANY WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT
OF COPYRIGHT, PATENT, TRADEMARK, OR OTHER RIGHT.  IN NO EVENT SHALL THE
COPYRIGHT HOLDER BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
INCLUDING ANY GENERAL, SPECIAL, INDIRECT, INCIDENTAL, OR CONSEQUENTIAL
DAMAGES, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF THE USE OR INABILITY TO USE THE FONT SOFTWARE OR FROM OTHER
DEALINGS IN THE FONT SOFTWARE.


```

## File: static\src\font\ocrb-license.txt

```text
Files: ocrb.otf
Copyright: 2012 Matthew Skala
License: public-domain
  This file is released to the public domain by its author, Matthew Skala.

```

## File: views\account_invoice.xml

```xml
<?xml version="1.0"?>
<odoo>
    <template id="l10n_ch_report_invoice_document" inherit_id="account.report_invoice_document">
        <xpath expr="//t[@t-set='show_qr']" position="attributes">
            <attribute name="t-value" add="and o.qr_code_method != 'ch_qr'" separator=" "/>
        </xpath>
    </template>
</odoo>

```

## File: views\account_payment_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="l10n_ch_account_payment_form" model="ir.ui.view">
            <field name="name">l10n_ch.account.payment.form</field>
            <field name="model">account.payment</field>
            <field name="inherit_id" ref="account.view_account_payment_form"/>
            <field name="arch" type="xml">
                <header position="after">
                    <div class="alert alert-warning" role="alert" invisible="not l10n_ch_reference_warning_msg">
                        <field name="l10n_ch_reference_warning_msg"/>
                    </div>
                </header>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\qr_invoice_wizard_view.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <record id="l10n_ch_qr_invoice_wizard_form" model="ir.ui.view">
        <field name="name">l10n_ch.qr_invoice.wizard.form</field>
        <field name="model">l10n_ch.qr_invoice.wizard</field>
        <field name="arch" type="xml">
            <form string="QR printing encountered a problem">
                <field name='nb_qr_inv' invisible="1"/>
                <field name="nb_classic_inv" invisible="1"/>
                <p>
                    <field name="qr_inv_text"/>
                    <field name="classic_inv_text"/>
                </p>
                <p>
                    To be able to print all invoices in the QR format, you might need to : <br/>
                    - check the account is a valid QR-IBAN<br/>
                    - or check your company and the partners are located in Switzerland.<br/>
                    Press Check Invalid Invoices to see a list of the invoices that were printed without a QR.
                </p>
                <footer>
                    <button name="print_all_invoices" string="Print All" type="object" class="btn-primary"/>
                    <button name="action_view_faulty_invoices" string="Check invalid invoices" type="object"/>
                </footer>
            </form>
        </field>
    </record>

    <record id="l10n_ch_qr_invoice_wizard" model="ir.actions.act_window">
        <field name="name">Qr Batch error Wizard</field>
        <field name="res_model">l10n_ch.qr_invoice.wizard</field>
        <field name="view_mode">form</field>
        <field name="view_id" ref="l10n_ch_qr_invoice_wizard_form"/>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: views\res_bank_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="isr_partner_bank_form" model="ir.ui.view">
            <field name="name">l10n_ch.res.partner.bank.form</field>
            <field name="model">res.partner.bank</field>
            <field name="inherit_id" ref="base.view_partner_bank_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='acc_number']" position="after">
                    <field name="l10n_ch_qr_iban" invisible="not l10n_ch_display_qr_bank_options"/>
                    <field name="l10n_ch_display_qr_bank_options" invisible="1"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\setup_wizard_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="setup_bank_account_wizard_inherit" model="ir.ui.view">
            <field name="name">account.setup.bank.manual.config.form.ch.inherit</field>
            <field name="model">account.setup.bank.manual.config</field>
            <field name="inherit_id" ref="account.setup_bank_account_wizard"/>
            <field name="arch" type="xml">
                <field name="bank_bic" position="after">
                    <field name="l10n_ch_display_qr_bank_options" invisible="1"/>
                    <field name="l10n_ch_qr_iban" invisible="not l10n_ch_display_qr_bank_options"/>
                </field>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\qr_invoice_wizard.py

```python
# -*- coding: utf-8 -*-
from odoo import models, fields, api, _
from odoo.exceptions import UserError


class QrInvoiceWizard(models.TransientModel):
    '''
    Wizard :
    When multiple invoices are selected to be printed in the QR-Iban format,
    this wizard will appear if one or more invoice(s) could not be QR-printed (wrong format...)
    The user will then be able to print the invoices that couldn't be printed in the QR format in the normal format, or
    to see a list of those.
    The non-QR invoices will have a note logged in their chatter, detailing the reason of the failure.
    '''
    _name = 'l10n_ch.qr_invoice.wizard'
    _description = 'Handles problems occurring while creating multiple QR-invoices at once'

    nb_qr_inv = fields.Integer(readonly=True)
    nb_classic_inv = fields.Integer(readonly=True)
    qr_inv_text = fields.Text(readonly=True)
    classic_inv_text = fields.Text(readonly=True)

    @api.model
    def default_get(self, fields):
        # Extends 'base'.

        def determine_invoices_text(nb_inv, inv_format):
            '''
            Creates a sentence explaining nb_inv invoices could be printed in the inv_format format.
            '''
            if nb_inv == 0:
                return _("No invoice could be printed in the %s format.", inv_format)
            if nb_inv == 1:
                return _("One invoice could be printed in the %s format.", inv_format)
            return _("%s invoices could be printed in the %s format.", nb_inv, inv_format)

        if not self._context.get('active_ids'):
            raise UserError(_("No invoice was found to be printed."))

        invoices = self.env['account.move'].browse(self._context['active_ids'])
        companies = invoices.company_id
        if len(companies) != 1 or companies[0].country_code != 'CH':
            raise UserError(_("All selected invoices must belong to the same Switzerland company"))

        results = super().default_get(fields)
        dispatched_invoices = invoices._l10n_ch_dispatch_invoices_to_print()
        results.update({
            'nb_qr_inv': len(dispatched_invoices['qr']),
            'nb_classic_inv': len(dispatched_invoices['classic']),
            'qr_inv_text': determine_invoices_text(nb_inv=len(dispatched_invoices['qr']), inv_format="QR"),
            'classic_inv_text': determine_invoices_text(nb_inv=len(dispatched_invoices['classic']), inv_format="classic"),
        })
        return results

    def print_all_invoices(self):
        '''
        Triggered by the Print All button
        '''
        all_invoices_ids = self.env.context.get('inv_ids')
        return self.env.ref('account.account_invoices').report_action(all_invoices_ids)

    def action_view_faulty_invoices(self):
        '''
        Open a list view of all the invoices that could not be printed in the QR format.
        '''
        # Prints the error stopping the invoice from being QR-printed in the invoice's chatter.
        invoices = self.env['account.move'].browse(self._context['active_ids'])
        dispatched_invoices = invoices._l10n_ch_dispatch_invoices_to_print()
        faulty_invoices = dispatched_invoices['classic']

        # Log a message inside the chatter explaining why the invoice is faulty.
        for inv in faulty_invoices:
            error_msg = inv.partner_bank_id._get_error_messages_for_qr('ch_qr', inv.partner_id, inv.currency_id)
            if error_msg:
                inv.message_post(body=error_msg, message_type="comment")
        action_vals = {
            'name': _("Invalid Invoices"),
            'type': 'ir.actions.act_window',
            'res_model': 'account.move',
            'context': {'create': False},
        }
        if len(faulty_invoices) == 1:
            action_vals.update({
                'view_mode': 'form',
                'res_id': faulty_invoices.id,
            })
        else:
            action_vals.update({
                'view_mode': 'tree',
                'domain': [('id', 'in', faulty_invoices.ids)],
            })
        return action_vals

```

## File: wizard\setup_wizards.py

```python
# -*- coding: utf-8 -*-
from odoo import api, models, fields


class SwissSetupBarBankConfigWizard(models.TransientModel):
    _inherit = 'account.setup.bank.manual.config'

    @api.onchange('acc_number')
    def _onchange_recompute_qr_iban(self):
        # Needed because ORM doesn't properly call the compute in 'new' mode, due to inherits, and
        # we want this field to be displayed in the wizard. We need to manually set acc_number
        # on the inherits m2o before calling the compute function manually.
        self.res_partner_bank_id.acc_number = self.acc_number
        self.res_partner_bank_id._compute_l10n_ch_qr_iban()
        self.l10n_ch_qr_iban = self.res_partner_bank_id.l10n_ch_qr_iban

    l10n_ch_display_qr_bank_options = fields.Boolean(compute='_compute_l10n_ch_display_qr_bank_options')

    @api.depends('partner_id', 'company_id')
    def _compute_l10n_ch_display_qr_bank_options(self):
        for wizard in self:
            wizard.l10n_ch_display_qr_bank_options = wizard.res_partner_bank_id.l10n_ch_display_qr_bank_options

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-

from . import setup_wizards
from . import qr_invoice_wizard

```

