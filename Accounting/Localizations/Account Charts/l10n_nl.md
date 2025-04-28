# Odoo Module: l10n_nl

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2016 Onestein (<http://www.onestein.eu>).

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Netherlands - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['nl'],
    'version': '3.4',
    'category': 'Accounting/Localizations/Account Charts',
    'author': 'Onestein (http://www.onestein.eu)',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/netherlands.html',
    'depends': [
        'base_iban',
        'base_vat',
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'data/account_account_tag.xml',
        'data/account_tax_report_data.xml',
        'views/res_config_settings_view.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'installable': True,
    'license': 'LGPL-3',
}

```

## File: data\account_account_tag.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">

        <!-- Account Tags -->
        <record id="account_tag_1" model="account.account.tag">
            <field name="name">Wages and salaries</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_2" model="account.account.tag">
            <field name="name">Accommodation costs</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
            <field name="active" eval="False"/>
        </record>
        <record id="account_tag_3" model="account.account.tag">
            <field name="name">Financial fixed assets</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
            <field name="active" eval="False"/>
        </record>
        <record id="account_tag_4" model="account.account.tag">
            <field name="name">Stocks</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_5" model="account.account.tag">
            <field name="name">Office costs</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_6" model="account.account.tag">
            <field name="name">Transport costs</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_7" model="account.account.tag">
            <field name="name">Cost of sales</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_8" model="account.account.tag">
            <field name="name">General expenses</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_9" model="account.account.tag">
            <field name="name">Shareholders' equity</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
            <field name="active" eval="False"/>
        </record>
        <record id="account_tag_10" model="account.account.tag">
            <field name="name">Cost of Goods Sold</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_11" model="account.account.tag">
            <field name="name">Net turnover</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
            <field name="active" eval="False"/>
        </record>
        <record id="account_tag_12" model="account.account.tag">
            <field name="name">Result on other assets</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
            <field name="active" eval="False"/>
        </record>
        <record id="account_tag_13" model="account.account.tag">
            <field name="name">Other current liabilities</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
            <field name="active" eval="False"/>
        </record>
        <record id="account_tag_14" model="account.account.tag">
            <field name="name">Provisions</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_15" model="account.account.tag">
            <field name="name">Non-current liabilities</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
            <field name="active" eval="False"/>
        </record>
        <record id="account_tag_16" model="account.account.tag">
            <field name="name">Other operating income</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_17" model="account.account.tag">
            <field name="name">Intangible assets</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
            <field name="active" eval="False"/>
        </record>
        <record id="account_tag_18" model="account.account.tag">
            <field name="name">Social charges</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_19" model="account.account.tag">
            <field name="name">Other personnel costs</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_20" model="account.account.tag">
            <field name="name">Pension charges</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_21" model="account.account.tag">
            <field name="name">Depreciation of property, plant and equipment</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_22" model="account.account.tag">
            <field name="name">Amortisation of intangible assets</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_23" model="account.account.tag">
            <field name="name">Property, plant and equipment</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_24" model="account.account.tag">
            <field name="name">Securities</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
            <field name="active" eval="False"/>
        </record>
        <record id="account_tag_25" model="account.account.tag">
            <field name="name">Cash at bank and in hand</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_26" model="account.account.tag">
            <field name="name">Distribution costs</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_27" model="account.account.tag">
            <field name="name">Receivables</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_28" model="account.account.tag">
            <field name="name">Suspense accounts</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
            <field name="active" eval="False"/>
        </record>
        <record id="account_tag_29" model="account.account.tag">
            <field name="name">Error account</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_30" model="account.account.tag">
            <field name="name">Interest income</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_31" model="account.account.tag">
            <field name="name">Interest and other financial charges</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_32" model="account.account.tag">
            <field name="name">Taxes &#38; social charges</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
            <field name="active" eval="False"/>
        </record>
        <record id="account_tag_33" model="account.account.tag">
            <field name="name">Taxes</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>

        <!-- Tags for BS tags report -->
        <record id="account_tag_immat" model="account.account.tag">
            <field name="name">Intangible Fixed Asset</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_fin" model="account.account.tag">
            <field name="name">Financial Fixed Active</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_cap" model="account.account.tag">
            <field name="name">Paid-up and Called-up Capital</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_premium" model="account.account.tag">
            <field name="name">Premium</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_revaluation_res" model="account.account.tag">
            <field name="name">Revaluation reserve</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_legal_res" model="account.account.tag">
            <field name="name">Legal and Statutory Reserves</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_other_res" model="account.account.tag">
            <field name="name">Other Reserves</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_undist_profit" model="account.account.tag">
            <field name="name">Undistributed Profit</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_prev_years_earnings" model="account.account.tag">
            <field name="name">Previous Years Profit</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_long_debt" model="account.account.tag">
            <field name="name">Long-term liabilities</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_sh_debt" model="account.account.tag">
            <field name="name">Current Liabilities</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_cred" model="account.account.tag">
            <field name="name">Creditors</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>

        <!-- Tags for PL tags report -->
        <record id="account_tag_net" model="account.account.tag">
            <field name="name">Net sales</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_hous" model="account.account.tag">
            <field name="name">Housing costs</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
        <record id="account_tag_reso" model="account.account.tag">
            <field name="name">Result other assets</field>
            <field name="applicability">accounts</field>
            <field name="country_id" ref="base.nl"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.nl"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="integer_rounding">DOWN</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_rub_1" model="account.report.line">
                <field name="name">Section 1: Domestic performance</field>
                <field name="aggregation_formula">1A_OMZET.balance + 1B_OMZET.balance + 1C_OMZET.balance + 1D_OMZET.balance + 1E_OMZET.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_rub_1a" model="account.report.line">
                        <field name="name">1a. Supplies/services taxed at high rate (turnover)</field>
                        <field name="code">1A_OMZET</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_1a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1a (omzet)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_1b" model="account.report.line">
                        <field name="name">1b. Supplies/services taxed at low rate (turnover)</field>
                        <field name="code">1B_OMZET</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_1b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1b (omzet)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_1c" model="account.report.line">
                        <field name="name">1c. Supplies/services taxed at other rates except 0% (turnover)</field>
                        <field name="code">1C_OMZET</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_1c_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1c (omzet)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_1d" model="account.report.line">
                        <field name="name">1d. Private use (turnover)</field>
                        <field name="code">1D_OMZET</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_1d_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1d (omzet)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_1e" model="account.report.line">
                        <field name="name">1e. Supplies/services taxed at 0% or not taxed with you (turnover)</field>
                        <field name="code">1E_OMZET</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_1e_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1e (omzet)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_rub_2" model="account.report.line">
                <field name="name">Section 2: Domestic reverse charge arrangements (turnover)</field>
                <field name="aggregation_formula">2A_OMZET.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_rub_2a" model="account.report.line">
                        <field name="name">2a. Supplies/services where the collection of turnover tax has been transferred to you (turnover)</field>
                        <field name="code">2A_OMZET</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_2a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2a (omzet)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_rub_3" model="account.report.line">
                <field name="name">Section 3: Performance to or from abroad (turnover)</field>
                <field name="aggregation_formula">3A_OMZET.balance + 3B_OMZET.balance + 3C_OMZET.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_rub_3a" model="account.report.line">
                        <field name="name">3a. Deliveries to countries outside the EU (exports) (turnover)</field>
                        <field name="code">3A_OMZET</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_3a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3a (omzet)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_3b" model="account.report.line">
                        <field name="name">3b. Deliveries to/services in countries within the EU and A-B-C supply chain transactions (turnover)</field>
                        <field name="code">3B_OMZET</field>
                        <field name="aggregation_formula">3BG_OMZET.balance + 3BT_OMZET.balance + 3BS_OMZET.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_rub_3bg" model="account.report.line">
                                <field name="name">3bg. Deliveries of products to countries within the EU (turnover).</field>
                                <field name="code">3BG_OMZET</field>
                                <field name="expression_ids">
                                    <record id="tax_report_rub_3bg_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3bg (omzet)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_rub_3bt" model="account.report.line">
                                <field name="name">3bt. A-B-C supply chain transactions within the EU (turnover)</field>
                                <field name="code">3BT_OMZET</field>
                                <field name="expression_ids">
                                    <record id="tax_report_rub_3bt_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3bt (omzet)</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_rub_3bs" model="account.report.line">
                                <field name="name">3bs. Services in countries within the EU (revenue)</field>
                                <field name="code">3BS_OMZET</field>
                                <field name="expression_ids">
                                    <record id="tax_report_rub_3bs_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3bs (omzet)</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_3c" model="account.report.line">
                        <field name="name">3c. Installation/remote sales within the EU (turnover)</field>
                        <field name="code">3C_OMZET</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_3c_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3c (omzet)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_rub_4" model="account.report.line">
                <field name="name">Section 4: Services provided to you from abroad (turnover)</field>
                <field name="aggregation_formula">4A_OMZET.balance + 4B_OMZET.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_rub_4a" model="account.report.line">
                        <field name="name">4a. Supplies/services from countries outside the EU (imports) (turnover)</field>
                        <field name="code">4A_OMZET</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_4a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4a (omzet)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_4b" model="account.report.line">
                        <field name="name">4b. Supplies/services from countries within the EU (turnover)</field>
                        <field name="code">4B_OMZET</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_4b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4b (omzet)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_rub_btw_1" model="account.report.line">
                <field name="name">Section 1: Domestic services (VAT)</field>
                <field name="code">NLTAX_B1</field>
                <field name="aggregation_formula">1A_BTW.balance + 1B_BTW.balance + 1C_BTW.balance + 1D_BTW.balance + 1E_BTW.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_rub_btw_1a" model="account.report.line">
                        <field name="name">1a. Supplies/services taxed at 21% (VAT)</field>
                        <field name="code">1A_BTW</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_1a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1a (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_1b" model="account.report.line">
                        <field name="name">1b. Supplies/services taxed at low rate (VAT)</field>
                        <field name="code">1B_BTW</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_1b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1b (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_1c" model="account.report.line">
                        <field name="name">1c. Supplies/services taxed at other rates except 0% (VAT)</field>
                        <field name="code">1C_BTW</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_1c_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1c (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_1d" model="account.report.line">
                        <field name="name">1d. Private use (VAT)</field>
                        <field name="code">1D_BTW</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_1d_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1d (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_1e" model="account.report.line">
                        <field name="name">1e. Supplies/services taxed at 0% or not taxed with you (VAT)</field>
                        <field name="code">1E_BTW</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_1e_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1e (BTW)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_rub_btw_2" model="account.report.line">
                <field name="name">Section 2: Domestic reverse charge (VAT) schemes</field>
                <field name="aggregation_formula">NLTAX_B2.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_rub_btw_2a" model="account.report.line">
                        <field name="name">2a. Supplies/services where the levy of Sales Tax has been transferred to you (VAT)</field>
                        <field name="code">NLTAX_B2</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_2a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">2a (BTW)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_rub_btw_4" model="account.report.line">
                <field name="name">Section 4: Services provided to you from abroad (VAT)</field>
                <field name="aggregation_formula">NLTAX_B4a.balance + NLTAX_B4b.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_rub_btw_4a" model="account.report.line">
                        <field name="name">4a. Supplies/services from countries outside the EU (VAT)</field>
                        <field name="code">NLTAX_B4a</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_4a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4a (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_4b" model="account.report.line">
                        <field name="name">4b. Supplies/services from countries within the EU (VAT)</field>
                        <field name="code">NLTAX_B4b</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_4b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4b (BTW)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_rub_btw_5" model="account.report.line">
                <field name="name">Section 5: Input tax, small business scheme and total (VAT)</field>
                <field name="aggregation_formula"></field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_rub_btw_5a" model="account.report.line">
                        <field name="name">5a. Sales tax payable (headings 1a to 4b) (VAT)</field>
                        <field name="aggregation_formula">NLTAX_B1.balance + NLTAX_B2.balance + NLTAX_B4a.balance + NLTAX_B4b.balance</field>
                    </record>
                    <record id="tax_report_rub_btw_5b" model="account.report.line">
                        <field name="name">5b. Input tax (VAT)</field>
                        <field name="code">NLTAX_B5b</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_5b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5b (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_5c" model="account.report.line">
                        <field name="name">5c. Subtotal (heading 5a minus 5b) (VAT)</field>
                        <field name="aggregation_formula">NLTAX_B1.balance + NLTAX_B2.balance + NLTAX_B4a.balance + NLTAX_B4b.balance - NLTAX_B5b.balance</field>
                    </record>
                    <record id="tax_report_rub_btw_5d" model="account.report.line">
                        <field name="name">5d. Reduction according to the small business scheme (VAT)</field>
                        <field name="code">NLTAX_B5d</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_5d_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5d. (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_5e" model="account.report.line">
                        <field name="name">5e. Estimate previous return(s) (VAT)</field>
                        <field name="code">NLTAX_B5e</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_5e_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5e. (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_5f" model="account.report.line">
                        <field name="name">5f. Estimate this return (VAT)</field>
                        <field name="code">NLTAX_B5f</field>
                        <field name="expression_ids">
                            <record id="tax_report_rub_btw_5f_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5f. (BTW)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_rub_btw_5g" model="account.report.line">
                        <field name="name">5g. Total payable/reclaimable (VAT)</field>
                        <field name="code">NLTAX_B5g</field>
                        <field name="aggregation_formula">NLTAX_B1.balance + NLTAX_B2.balance + NLTAX_B4a.balance + NLTAX_B4b.balance - NLTAX_B5b.balance - NLTAX_B5d.balance - NLTAX_B5e.balance - NLTAX_B5f.balance</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-nl.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@nl","name@de"
"0010","Acquisition value Goodwill","0010","asset_fixed","l10n_nl.account_tag_immat","False","Aanschafwaarde Goodwill","Anschaffungswert Geschäftswert"
"0019","Amortisation Goodwill","0019","asset_fixed","l10n_nl.account_tag_immat","False","Afschrijving Goodwill","Amortisation Geschäftswert"
"0020","Acquisition value other intangible assets","0020","asset_fixed","l10n_nl.account_tag_immat","False","Aanschafwaarde overige immateriele vaste activa","Anschaffungswert sonstige immaterielle Vermögenswerte"
"0029","Amortisation of other intangible assets","0029","asset_fixed","l10n_nl.account_tag_immat","False","Afschrijving overige immateriele vaste activa","Amortisation auf sonstige immaterielle Vermögenswerte"
"0101","Land purchase value","0101","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Grond","Wert des Grunderwerbs"
"0110","Acquisition value of buildings","0110","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Gebouwen","Anschaffungswert der Gebäude"
"0119","Depreciation of buildings","0119","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving Gebouwen","Abschreibung von Gebäuden"
"0120","Acquisition value of shops","0120","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Winkels","Anschaffungswert der Geschäfte"
"0129","Depreciation of shops","0129","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving Winkels","Abschreibung von Geschäften"
"0130","Acquisition value of renovations","0130","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Verbouwingen","Anschaffungswert der Renovierungen"
"0139","Depreciation of renovations","0139","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving Verbouwingen","Abschreibung von Renovierungen"
"0210","Acquisition value of machinery 1","0210","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Machines 1","Anschaffungswert der Maschinen 1"
"0219","Depreciation of machinery 1","0219","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving Machines 1","Abschreibung von Maschinen 1"
"0220","Acquisition value of machinery 2","0220","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Machines 2","Anschaffungswert der Maschinen 2"
"0229","Depreciation of machinery 2","0229","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving Machines 2","Abschreibung von Maschinen 2"
"0310","Purchase value of business equipment","0310","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Bedrijfsinventaris","Anschaffungswert der Geschäftsausstattung"
"0319","Depreciation of business equipment","0319","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving Bedrijfsinventaris","Abschreibung von Geschäftsausstattung"
"0320","Purchase value of warehouse inventory","0320","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Magazijninventaris","Anschaffungswert des Lagerbestands"
"0329","Depreciation of warehouse inventory","0329","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving Magazijninventaris","Abschreibung von Lagerbeständen"
"0330","Purchase value of factory inventory","0330","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Fabrieksinventaris","Abschreibung des Fabrikinventars"
"0339","Depreciation of factory inventory","0339","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving Fabrieksinventaris","Abschreibung des Fabrikinventars"
"0340","Purchase value of tools","0340","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Gereedschappen","Anschaffungswert der Werkzeuge"
"0349","Depreciation of tools","0349","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving Gereedschappen","Abschreibung von Werkzeugen"
"0350","Purchase value of office equipment","0350","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Kantoorinventaris","Anschaffungswert der Büroausstattung"
"0359","Depreciation of office equipment","0359","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving Kantoorinventaris","Abschreibung von Büromaterial"
"0360","Purchase value of office machinery","0360","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Kantoormachines","Anschaffungswert der Büromaschinen"
"0369","Depreciation of office machinery","0369","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving Kantoormachines","Abschreibung von Büromaschinen"
"0370","Purchase value of canteen inventory","0370","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Kantine-inventaris","Anschaffungswert des Kantineninventars"
"0379","Depreciation of canteen inventory","0379","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving Kantine-inventaris","Abschreibung des Kantineninventars"
"0410","Purchase value of passenger cars","0410","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Personenauto's","Anschaffungswert von Personenkraftwagen"
"0419","Depreciation of passenger cars","0419","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving Personenauto's","Abschreibung von Personenkraftwagen"
"0420","Purchase value other transport equipment","0420","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde overige vervoersmiddelen","Anschaffungswert sonstige Transportmittel"
"0429","Depreciation of other transport equipment","0429","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving overige vervoersmiddelen","Abschreibung von sonstigem Transportmaterial"
"0430","Purchase value of trucks","0430","asset_fixed","l10n_nl.account_tag_23","False","Aanschafwaarde Vrachtauto's","Abschreibung von Lastkraftwagen"
"0439","Depreciation of trucks","0439","asset_fixed","l10n_nl.account_tag_23","False","Afschrijving Vrachtauto's","Abschreibung von Lastkraftwagen"
"0510","Participation 1","0510","asset_fixed","l10n_nl.account_tag_fin","False","Deelneming 1","Teilnahme 1"
"0520","Participation 2","0520","asset_fixed","l10n_nl.account_tag_fin","False","Deelneming 2","Teilnahme 2"
"0530","Participation 3","0530","asset_fixed","l10n_nl.account_tag_fin","False","Deelneming 3","Teilnahme 3"
"0551","Receivables from participation 1","0551","asset_fixed","l10n_nl.account_tag_fin","False","Vorderingen op deelneming 1","Forderungen aus Beteiligungen 1"
"0552","Receivables from participation 2","0552","asset_fixed","l10n_nl.account_tag_fin","False","Vorderingen op deelneming 2","Forderungen aus Beteiligungen 2"
"0553","Receivables from participation 3","0553","asset_fixed","l10n_nl.account_tag_fin","False","Vorderingen op deelneming 3","Forderungen aus Beteiligungen 3"
"0561","Mortgages receivable 1","0561","asset_fixed","l10n_nl.account_tag_fin","False","Hypotheken u/g 1","Forderungen aus Hypotheken 1"
"0562","Mortgages receivable 2","0562","asset_fixed","l10n_nl.account_tag_fin","False","Hypotheken u/g 2","Forderungen aus Hypotheken 2"
"0563","Mortgages receivable 3","0563","asset_fixed","l10n_nl.account_tag_fin","False","Hypotheken u/g 3","Forderungen aus Hypotheken 3"
"0565","Loans receivable 1","0565","asset_fixed","l10n_nl.account_tag_fin","False","Leningen u/g 1","Forderungen aus Darlehen 1"
"0566","Loans receivable 2","0566","asset_fixed","l10n_nl.account_tag_fin","False","Leningen u/g 2","Forderungen aus Darlehen 2"
"0567","Loans receivable 3","0567","asset_fixed","l10n_nl.account_tag_fin","False","Leningen u/g 3","Forderungen aus Darlehen 3"
"0570","Security deposits","0570","asset_fixed","l10n_nl.account_tag_fin","False","Waarborgsommen","Kautionen"
"0610","Share capital","0610","equity","l10n_nl.account_tag_cap","False","Aandelenkapitaal","Grundkapital"
"0620","Legal reserves","0620","equity","l10n_nl.account_tag_legal_res","False","Wettelijke reserves","Gesetzliche Reserven"
"0630","Share premium reserve","0630","equity","l10n_nl.account_tag_premium","False","Agioreserve","Kapitalrücklage"
"0640","General reserve","0640","equity","l10n_nl.account_tag_other_res","False","Algemene reserve","Allgemeine Reserve"
"0650","Other reserves","0650","equity","l10n_nl.account_tag_other_res","False","Overige reserves","Sonstige Rücklagen"
"0660","Revaluation reserve","0660","equity","l10n_nl.account_tag_revaluation_res","False","Herwaarderingsreserve","Neubewertungsrücklage"
"0710","Provision for pension liabilities","0710","liability_non_current","l10n_nl.account_tag_14","False","Voorziening pensioenverplichtingen","Rückstellung für Pensionsverpflichtungen"
"0720","Major maintenance equalisation reserve","0720","liability_non_current","l10n_nl.account_tag_14","False","Egalisatiereserve groot onderhoud","Schwankungsreserve für größere Instandhaltungsmaßnahmen"
"0730","Provision for participating interests","0730","liability_non_current","l10n_nl.account_tag_14","False","Voorziening deelnemingen","Rückstellung für Beteiligungen"
"0740","Deferred tax liability","0740","liability_non_current","l10n_nl.account_tag_14","False","Latente belastingverplichting","Latente Steuerschuld"
"0750","Other provisions","0750","liability_non_current","l10n_nl.account_tag_14","False","Overige voorzieningen","Sonstige Bestimmungen"
"0760","Guarantee liabilities","0760","liability_non_current","l10n_nl.account_tag_14","False","Garantieverplichtingen","Bürgschaftsverpflichtungen"
"0811","Mortgages payable 1","0811","liability_non_current","l10n_nl.account_tag_long_debt","False","Hypotheken o/g 1","Verbindlichkeiten aus Hypotheken 1"
"0812","Mortgages payable 2","0812","liability_non_current","l10n_nl.account_tag_long_debt","False","Hypotheken o/g 2","Verbindlichkeiten aus Hypotheken 2"
"0813","Mortgages payable 3","0813","liability_non_current","l10n_nl.account_tag_long_debt","False","Hypotheken o/g 3","Verbindlichkeiten aus Hypotheken 3"
"0814","Mortgages payable 4","0814","liability_non_current","l10n_nl.account_tag_long_debt","False","Hypotheken o/g 4","Verbindlichkeiten aus Hypotheken 4"
"0815","Mortgages payable 5","0815","liability_non_current","l10n_nl.account_tag_long_debt","False","Hypotheken o/g 5","Verbindlichkeiten aus Hypotheken 5"
"0821","Loans payable 1","0821","liability_non_current","l10n_nl.account_tag_long_debt","False","Leningen o/g 1","Verbindlichkeiten aus Krediten 1"
"0822","Loans payable 2","0822","liability_non_current","l10n_nl.account_tag_long_debt","False","Leningen o/g 2","Verbindlichkeiten aus Krediten 2"
"0823","Loans payable 3","0823","liability_non_current","l10n_nl.account_tag_long_debt","False","Leningen o/g 3","Verbindlichkeiten aus Krediten 3"
"0824","Loans payable 4","0824","liability_non_current","l10n_nl.account_tag_long_debt","False","Leningen o/g 4","Verbindlichkeiten aus Krediten 4"
"0825","Loans payable 5","0825","liability_non_current","l10n_nl.account_tag_long_debt","False","Leningen o/g 5","Verbindlichkeiten aus Krediten 5"
"1040","Demand items","1040","asset_cash","l10n_nl.account_tag_25","False","Vraagposten","Bedarfspositionen"
"recv","Debtors","1100","asset_receivable","l10n_nl.account_tag_27","True","Debiteuren","Debitoren"
"recv_pos","Debtors (PoS)","1101","asset_receivable","l10n_nl.account_tag_27","True","Debiteuren (PoS)","Debitoren (PoS)"
"1150","Doubtful debtors","1150","asset_current","l10n_nl.account_tag_27","False","Dubieuze debiteuren","Zweifelhafte Schuldner"
"1160","Provision for doubtful debts","1160","asset_current","l10n_nl.account_tag_27","False","Voorziening dubieuze debiteuren","Rückstellung für zweifelhafte Forderungen"
"1205","Prepaid expenses","1205","asset_prepayments","l10n_nl.account_tag_27","False","Vooruitbetaalde kosten","Aktive Rechnungsabgrenzungsposten"
"1210","Deposit","1210","asset_prepayments","l10n_nl.account_tag_27","False","Borg","Einzahlung"
"1220","Interest to be received","1220","asset_current","l10n_nl.account_tag_27","False","Te ontvangen rente","Zu erhaltende Zinsen"
"1230","Sick pay to be received","1230","asset_current","l10n_nl.account_tag_27","False","Te ontvangen ziekengeld","Zu erhaltendes Krankengeld"
"1250","Still to be invoiced (outgoing stock)","1250","asset_current","l10n_nl.account_tag_27","False","Nog te factureren (uitgaande voorraad)","Noch zu fakturieren (Lagerabgang)"
"1290","Other short-term receivables","1290","asset_current","l10n_nl.account_tag_27","False","Overige kortlopende vorderingen","Sonstige kurzfristige Forderungen"
"pay","Creditors","1300","liability_payable","l10n_nl.account_tag_cred","True","Crediteuren","Gläubiger"
"1350","Payments in transit","1350","liability_current","l10n_nl.account_tag_sh_debt","False","Betalingen onderweg","Durchlaufende Zahlungen"
"1405","Invoiced amounts in advance","1405","liability_current","l10n_nl.account_tag_sh_debt","False","Vooruit gefactureerde bedragen","Im Voraus in Rechnung gestellte Beträge"
"1410","Interest to be paid","1410","liability_current","l10n_nl.account_tag_sh_debt","False","Te betalen rente","Zu zahlende Zinsen"
"1420","Dividends to be paid","1420","liability_current","l10n_nl.account_tag_sh_debt","False","Te betalen dividend","Auszuschüttende Dividenden"
"1430","Audit fees to be paid","1430","liability_current","l10n_nl.account_tag_sh_debt","False","Te betalen accountantskosten","Zu zahlende Prüfungsgebühren"
"1440","Telephone charges to be paid","1440","liability_current","l10n_nl.account_tag_sh_debt","False","Te betalen telefoonkosten","Zu zahlende Telefongebühren"
"1450","Invoices to be received (incoming stock)","1450","liability_current","l10n_nl.account_tag_sh_debt","False","Nog te ontvangen facturen (inkomende voorraad)","Zu erhaltende Rechnungen (Eingangsbestand)"
"1460","Income tax to be paid","1460","liability_current","l10n_nl.account_tag_sh_debt","False","Te betalen VPB","Zu zahlende Einkommensteuer"
"1470","Income tax from previous years to be paid","1470","liability_current","l10n_nl.account_tag_sh_debt","False","Te betalen VPB voorgaande jaren","Zu zahlende Einkommensteuer aus früheren Jahren"
"1480","Dividend tax to be paid","1480","liability_current","l10n_nl.account_tag_sh_debt","False","Te betalen dividendbelasting","Zu zahlende Dividendensteuer"
"1490","Other current liabilities","1490","liability_current","l10n_nl.account_tag_sh_debt","False","Overige kortlopende schulden","Sonstige kurzfristige Verbindlichkeiten"
"vat_payable_h","Deferred VAT high rate","1500","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW hoog tarief","Aufgeschobene Mehrwertsteuer hoher Satz"
"vat_payable_l","Deferred VAT low rate","1501","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW laag tarief","Aufgeschobene Mehrwertsteuer niedriger Satz"
"vat_payable_0","Deferred VAT other rates","1502","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW overige tarieven","Aufgeschobene Mehrwertsteuer andere Sätze"
"1503","Deductible VAT on private use","1503","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW privé-gebruik","Abzugsfähige Mehrwertsteuer bei privater Nutzung"
"vat_payable_v","Chargeable reverse charge VAT","1504","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW heffing verlegd","Steuerpflichtige Verlagerung der Steuerschuldnerschaft"
"1505","VAT 0% / not taxed with me","1505","liability_current","l10n_nl.account_tag_sh_debt","False","BTW 0% / niet bij mij belast","Mehrwertsteuer 0% / bei mir nicht besteuert"
"1507","Installation/tele sales within the EU","1507","liability_current","l10n_nl.account_tag_sh_debt","False","Installatie/televerkoop binnen de EU","Installation/Telefonverkauf innerhalb der EU"
"1508","Deliveries made to me outside the EU","1508","liability_current","l10n_nl.account_tag_sh_debt","False","Aan mij verrichte leveringen buiten EU","Lieferungen an mich außerhalb der EU"
"1509","Deliveries within the EU made to me","1509","liability_current","l10n_nl.account_tag_sh_debt","False","Aan mij verrichte leveringen binnen EU","Lieferungen innerhalb der EU an mich"
"vat_refund_0","Pre-tax other","1510","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting overige","Sonstige vor Steuern"
"vat_payable_h_non_eu","Deferred VAT high rate outside the EU","1513","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW hoog tarief buiten de EU","Aufgeschobene Mehrwertsteuer hoher Satz außerhalb der EU"
"vat_payable_l_non_eu","Deferred VAT low rate outside the EU","1514","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW laag tarief buiten de EU","Aufgeschobener niedriger Mehrwertsteuersatz außerhalb der EU"
"vat_payable_0_non_eu","Deferred VAT other rates outside the EU","1515","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW overige tarieven buiten de EU","Aufgeschobene Mehrwertsteuer andere Sätze außerhalb der EU"
"vat_payable_h_eu","EU high rate VAT to be paid","1516","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW hoog tarief binnen de EU","Hoher EU-Mehrwertsteuersatz zu entrichten"
"vat_payable_l_eu","EU low rate VAT to be paid","1517","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW laag tarief binnen de EU","Niedriger EU-Mehrwertsteuersatz zu entrichten"
"vat_payable_0_eu","EU other rate VAT to be paid","1518","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW overige tarieven binnen de EU","EU anderer Satz zu zahlende Mehrwertsteuer"
"vat_refund_h","Pre-tax high","1520","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting hoog","Hoch vor Steuern"
"vat_refund_l","Pre-tax low","1521","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting laag","Niedrig vor Steuern"
"vat_refund_v","Pre-tax shifted","1522","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting verlegd","Vor Steuern verschoben"
"vat_refund_0_eu","Pre-tax other within the EU","1523","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting overige binnen de EU","Sonstige innerhalb der EU vor Steuern"
"vat_refund_h_eu","Pre-tax high within the EU","1524","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting hoog binnen de EU","Hohe Vorsteuerbeträge innerhalb der EU"
"vat_refund_l_eu","Pre-tax low within the EU","1525","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting laag binnen de EU","Niedriges Vorsteuerergebnis innerhalb der EU"
"vat_refund_v_eu","Pre-tax carried forward within the EU","1526","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting verlegd binnen de EU","Vorsteuer-Vortrag innerhalb der EU"
"vat_refund_0_non_eu","Pre-tax other outside EU","1527","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting overige buiten EU","Sonstige außerhalb der EU vor Steuern"
"vat_refund_h_non_eu","Pre-tax high outside EU","1528","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting hoog buiten EU","Hoch vor Steuern außerhalb der EU"
"vat_refund_l_non_eu","Pre-tax low outside EU","1529","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting laag buiten EU","Niedrig vor Steuern außerhalb der EU"
"vat_refund_v_non_eu","Pre-tax carried forward outside EU","1530","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting verlegd buiten EU","Vorsteuer-Vortrag außerhalb der EU"
"1531","VAT bad debts","1531","liability_current","l10n_nl.account_tag_sh_debt","False","BTW oninbare vorderingen","Uneinbringliche Forderungen bei der Mehrwertsteuer"
"1532","Small business regulation","1532","liability_current","l10n_nl.account_tag_sh_debt","False","Kleine ondernemersregeling","Regulierung kleiner Unternehmen"
"vat_refund_v_d_non_eu","Pre-tax reverse charge outside EU services","1535","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting verlegd buiten EU diensten","Umkehrung der Steuerschuldnerschaft außerhalb der EU Dienstleistungen"
"vat_payable_h_d","Deferred VAT high rate services","1540","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW hoog tarief diensten","Aufgeschobene MwSt. für Dienstleistungen mit hohem Steuersatz"
"vat_payable_l_d","Deferred VAT low rate services","1541","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW laag tarief diensten","Aufgeschobene MwSt. für niedrig besteuerte Dienstleistungen"
"vat_payable_0_d","Deferred VAT other rate services","1542","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW overige tarieven diensten","Aufgeschobene Mehrwertsteuer andere Dienstleistungen"
"vat_payable_v_d","Deferred VAT to be paid on reverse-charge services","1544","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW heffing verlegd diensten","Aufgeschobene Mehrwertsteuer auf Reverse-Charge-Dienstleistungen zu zahlen"
"1550","Sales tax payments/declaration","1550","liability_current","l10n_nl.account_tag_sh_debt","False","Afdrachten/aangifte omzetbelasting","Umsatzsteuerzahlungen/-erklärung"
"1560","Sales tax supplement","1560","liability_current","l10n_nl.account_tag_sh_debt","False","Suppletie omzetbelasting","Zuschlag für die Umsatzsteuer"
"vat_refund_0_d","Pre-tax other services","1570","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting overige diensten","Sonstige Dienstleistungen vor Steuern"
"vat_payable_h_d_non_eu","Deferred VAT high rate outside EU services","1573","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW hoog tarief buiten de EU diensten","Aufgeschobene MwSt. hoher Satz außerhalb der EU Dienstleistungen"
"vat_payable_l_d_non_eu","Deferred VAT low rate outside EU services","1574","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW laag tarief buiten de EU diensten","Aufgeschobener niedriger Mehrwertsteuersatz außerhalb der EU Dienstleistungen"
"vat_payable_0_d_non_eu","Deferred VAT other rates outside EU services","1575","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW overige tarieven buiten de EU diensten","Aufgeschobene Mehrwertsteuer andere Sätze außerhalb der EU Dienstleistungen"
"vat_payable_h_d_eu","Deferred VAT high rate within EU services","1576","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW hoog tarief binnen de EU diensten","Aufgeschobener hoher Mehrwertsteuersatz bei Dienstleistungen in der EU"
"vat_payable_l_d_eu","Deferred VAT low rate within EU services","1577","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW laag tarief binnen de EU diensten","Aufgeschobener niedriger Mehrwertsteuersatz für Dienstleistungen in der EU"
"vat_payable_0_d_eu","Deferred VAT other rate within EU services","1578","liability_current","l10n_nl.account_tag_sh_debt","False","Af te dragen BTW overige tarief binnen de EU diensten","Aufgeschobene Mehrwertsteuer anderer Satz innerhalb der EU Dienstleistungen"
"vat_refund_h_d","Pre-tax high services","1580","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting hoog diensten","Hohe Dienstleistungen vor Steuern"
"vat_refund_l_d","Pre-tax low services","1581","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting laag diensten","Niedrige Dienstleistungen vor Steuern"
"vat_refund_v_d","Pre-tax reverse charge services","1582","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting verlegd diensten","Reverse-Charge-Dienstleistungen vor Steuern"
"vat_refund_0_d_eu","Pre-tax other intra-EU services","1583","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting overige binnen de EU diensten","Sonstige Intra-EU-Dienstleistungen vor Steuern"
"vat_refund_h_d_eu","Pre-tax high within EU services","1584","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting hoog binnen de EU diensten","Hohe Vorsteuerbeträge im EU-Dienstleistungssektor"
"vat_refund_l_d_eu","Pre-tax low within EU services","1585","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting laag binnen de EU diensten","Niedriges Vorsteuerergebnis bei EU-Dienstleistungen"
"vat_refund_v_d_eu","Pre-tax reverse charge within EU services","1586","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting verlegd binnen de EU diensten","Verlagerung der Steuerschuldnerschaft innerhalb der EU-Dienstleistungen vor Steuern"
"vat_refund_0_d_non_eu","Pre-tax other non-EU services","1587","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting overige buiten EU diensten","Sonstige Nicht-EU-Dienstleistungen vor Steuern"
"vat_refund_h_d_non_eu","Pre-tax high outside EU services","1588","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting hoog buiten EU diensten","Hohe Vorsteuerbeträge für Dienstleistungen außerhalb der EU"
"vat_refund_l_d_non_eu","Pre-tax low outside EU services","1589","liability_current","l10n_nl.account_tag_sh_debt","False","Voorbelasting laag buiten EU diensten","Niedriges Vorsteuerergebnis außerhalb der EU Dienstleistungen"
"1599","Sales tax liability (end of financial year)","1599","liability_current","l10n_nl.account_tag_sh_debt","False","Schuld omzetbelasting (einde boekjaar)","Umsatzsteuerschuld (Ende des Haushaltsjahres)"
"1610","Pension premiums","1610","liability_current","l10n_nl.account_tag_sh_debt","False","Pensioenpremies","Rentenprämien"
"1615","Payment of pension contributions","1615","liability_current","l10n_nl.account_tag_sh_debt","False","Afdracht pensioenpremies","Zahlung von Rentenbeiträgen"
"1620","Net salaries","1620","liability_current","l10n_nl.account_tag_sh_debt","False","Netto lonen","Nettogehälter"
"1621","Advance payment of wages","1621","liability_current","l10n_nl.account_tag_sh_debt","False","Voorschot loon","Lohnvorauszahlung"
"1625","Net wages to be paid","1625","liability_current","l10n_nl.account_tag_sh_debt","False","Te betalen netto lonen","Zu zahlende Nettolöhne"
"1630","Payroll tax","1630","liability_current","l10n_nl.account_tag_sh_debt","False","Loonheffing","Lohnsummensteuer"
"1635","Remittance of wage tax","1635","liability_current","l10n_nl.account_tag_sh_debt","False","Afdracht loonheffing","Abführung der Lohnsteuer"
"1640","Holidays to be paid","1640","liability_current","l10n_nl.account_tag_sh_debt","False","Te betalen vakantiedagen","Zu bezahlende Feiertage"
"1650","Holiday pay due","1650","liability_current","l10n_nl.account_tag_sh_debt","False","Te betalen vakantiegeld","Fälliges Urlaubsgeld"
"2010","Opening balance sheet suspense account","2010","liability_current","l10n_nl.account_tag_sh_debt","False","Tussenrekening beginbalans","Eröffnungsbilanz-Spannungskonto"
"3001","Raw materials 1","3001","asset_current","l10n_nl.account_tag_4","False","Grondstoffen 1","Rohmaterialien 1"
"3002","Raw materials 2","3002","asset_current","l10n_nl.account_tag_4","False","Grondstoffen 2","Rohmaterialien 2"
"3100","Excipients 1","3100","asset_current","l10n_nl.account_tag_4","False","Hulpstoffen 1","Hilfsstoffe 1"
"3110","Excipients 2","3110","asset_current","l10n_nl.account_tag_4","False","Hulpstoffen 2","Hilfsstoffe 2"
"3200","Stock 1","3200","asset_current","l10n_nl.account_tag_4","False","Voorraad 1","Lagerbestand 1"
"3210","Stock 2","3210","asset_current","l10n_nl.account_tag_4","False","Voorraad 2","Lagerbestand 2"
"3300","Packaging material","3300","asset_current","l10n_nl.account_tag_4","False","Verpakkingsmateriaal","Verpackungsmaterial"
"3310","Packaging","3310","asset_current","l10n_nl.account_tag_4","False","Emballage","Verpackung"
"4001","Gross wages","4001","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_1","False","Bruto lonen","Bruttolöhne"
"4002","Bonuses and commissions","4002","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_1","False","Bonussen en provisies","Boni und Provisionen"
"4003","Holiday allowance","4003","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_1","False","Vakantietoeslag","Urlaubsgeld"
"4004","Royalty","4004","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_1","False","Tantièmes","Lizenzgebühren"
"4005","Employee car contribution","4005","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_1","False","Werknemersbijdrage auto","Beitrag der Mitarbeiter zum Auto"
"4006","Healthcare Insurance Act (SVW) contribution","4006","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_1","False","Bijdrage zorgverzekeringswet (SVW)","Krankenversicherungsgesetz (SVW) Beitrag"
"4010","Employer's share of payroll taxes","4010","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_1","False","Werkgeversdeel loonheffingen","Anteil des Arbeitgebers an der Lohnsummensteuer"
"4011","Employer's share of pensions","4011","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_20","False","Werkgeversdeel pensioenen","Anteil des Arbeitgebers an den Renten"
"4012","Employer's share of social security contributions","4012","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_18","False","Werkgeversdeel sociale lasten","Anteil des Arbeitgebers an den Sozialversicherungsbeiträgen"
"4015","Provision for holidays","4015","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_1","False","Voorziening vakantiedagen","Rückstellung für Urlaub"
"4016","Compensation for commuting","4016","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_1","False","Vergoeding woon-werkverkeer","Entschädigung für Pendler"
"4017","Reimbursement of study costs","4017","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_19","False","Vergoeding studiekosten","Rückerstattung der Studienkosten"
"4018","Reimbursement of other travel expenses","4018","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_19","False","Vergoeding overige reiskosten","Erstattung der sonstigen Reisekosten"
"4019","Reimbursement of other expenses","4019","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_19","False","Vergoeding overige kosten","Erstattung sonstiger Kosten"
"4020","Management fees","4020","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_19","False","Managementvergoedingen","Verwaltungsgebühren"
"4021","Staff on loan","4021","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_19","False","Ingeleend personeel","Ausgeliehenes Personal"
"4022","Working expenses scheme (WKR max 1.2% gross pay)","4022","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_19","False","Werkkostenregeling (WKR max 1.2% brutoloon)","Arbeitskostenregelung (WKR max. 1,2 % des Bruttolohns)"
"4024","Travel costs of hired staff","4024","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_19","False","Reiskosten ingeleend personeel","Reisekosten für eingestelltes Personal"
"4029","Recharge of direct labour costs","4029","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_19","False","Doorbelasting directe loonkosten","Weiterberechnung der direkten Arbeitskosten"
"4030","Sick leave insurance","4030","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_18","False","Ziekteverzuimverzekering","Versicherung gegen Krankheitsurlaub"
"4040","Canteen costs","4040","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_5","False","Kantinekosten","Kantinenkosten"
"4041","Corporate clothing","4041","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_5","False","Bedrijfskleding","Firmenkleidung"
"4042","Other travel expenses","4042","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_5","False","Overige reiskosten","Sonstige Reisekosten"
"4043","Conferences, seminars and symposia","4043","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_5","False","Vergoeding woon- werkverkeer","Konferenzen, Seminare und Symposien"
"4044","Staff recruitment costs","4044","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_19","False","Wervingskosten personeel","Kosten für die Einstellung von Personal"
"4045","Study and training costs","4045","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_19","False","Studie- en opleidingskosten","Studien- und Ausbildungskosten"
"4090","Other personnel costs","4090","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_19","False","Overige personeelskosten","Sonstige Personalkosten"
"4110","Property rental","4110","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Huur onroerend goed","Vermietung von Immobilien"
"4111","Major property maintenance","4111","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Groot onderhoud onroerend goed","Instandhaltung von Immobilien"
"4112","Property maintenance","4112","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Onderhoud onroerend goed","Instandhaltung von Immobilien"
"4113","Addition to large-scale maintenance equalisation reserve","4113","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Toevoeging egalisatiereserve Groot onderhoud","Zuführung zur Schwankungsreserve für Großanlagen"
"4120","Property taxes","4120","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Belastingen onroerend goed","Grundsteuern"
"4130","Energy costs","4130","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Energiekosten","Energiekosten"
"4140","Cleaning costs","4140","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Schoonmaakkosten","Kosten für die Reinigung"
"4150","Property insurance","4150","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Assurantie onroerend goed","Sachversicherung"
"4190","Other housing costs","4190","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Overige huisvestingskosten","Sonstige Wohnkosten"
"4201","Machine rental","4201","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Huur machines","Vermietung von Maschinen"
"4202","Machines leasen (operationele lease)","4202","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Lease machines (operational lease)","Geleaste Maschinen (Betriebsleasing)"
"4210","Inventory rental","4210","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Huur inventaris","Vermietung von Inventar"
"4211","Lease inventaris (operationele lease)","4211","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Lease inventaris (operational lease)","Leasing-Inventare (Operating-Leasing)"
"4220","Machinery maintenance","4220","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Onderhoud machines","Instandhaltung von Maschinen"
"4230","Inventory maintenance","4230","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Onderhoud inventaris","Pflege des Inventars"
"4240","Tools","4240","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Gereedschappen","Werkzeuge"
"4250","Small acquisitions","4250","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Kleine aanschaffingen","Kleine Akquisitionen"
"4260","Energy costs machines/inventory","4260","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Energiekosten machines / inventaris","Energiekosten Maschinen/Lagerbestand"
"4270","Insurance machinery/inventory","4270","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Assurantie machines / inventaris","Versicherung Maschinen/Vorräte"
"4280","Environmental costs","4280","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Milieukosten","Umweltkosten"
"4290","Other operating expenses","4290","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Overige bedrijfskosten","Sonstige betriebliche Aufwendungen"
"4305","Contributies / subscriptionen","4305","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_8","False","Contributies / abonnementen","Beiträge / Abonnements"
"4310","Office supplies","4310","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_5","False","Kantoorbenodigdheden","Büromaterial"
"4320","Maintenance of office equipment","4320","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_5","False","Onderhoud kantoorinventaris","Wartung der Büroausstattung"
"4321","Office equipment rental","4321","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_5","False","Huur kantoorapparatuur","Vermietung von Büroausstattung"
"4322","Lease of office equipment (operating lease)","4322","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_5","False","Lease kantoorinventaris (operational lease)","Leasing von Büroausstattung (Operating Lease)"
"4330","Phone","4330","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_5","False","Telefoon","Telefon"
"4340","Internet / e-mail","4340","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_5","False","Internet / e-mail","Internet/E-Mail"
"4350","Postage","4350","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_5","False","Porti","Porto"
"4360","Automation costs","4360","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_hous","False","Automatiseringskosten","Kosten der Automatisierung"
"4390","Other office expenses","4390","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_5","False","Overige kantoorkosten","Sonstige Bürokosten"
"4401","Fuel for trucks","4401","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Brandstof vrachtauto's","Kraftstoff für Lkw"
"4402","Truck maintenance","4402","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Onderhoud vrachtauto's","Wartung von Lastkraftwagen"
"4403","Truck rental","4403","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Huur vrachtauto's","Lkw-Vermietung"
"4404","Leasing trucks (operational lease)","4404","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Lease vrachtauto's (operational lease)","Leasing von Lastkraftwagen (Operational Lease)"
"4405","Insurance trucks","4405","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Assurantie vrachtauto's","Versicherung Lkw"
"4406","Truck taxes","4406","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Belastingen vrachtauto's","Lkw-Steuern"
"4408","Other costs of trucks","4408","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Overige kosten vrachtauto's","Sonstige Kosten für Lkw"
"4411","Fuel cars","4411","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Brandstof personenauto's","Benzinautos"
"4412","Car maintenance","4412","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Onderhoud personenauto's","Autopflege"
"4413","Car rental","4413","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Huur personenauto's","Autovermietung"
"4414","Lease of cars (operational lease)","4414","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Lease personenauto's (operational lease)","Leasing von Fahrzeugen (operatives Leasing)"
"4415","Car insurance","4415","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Assurantie personenauto's","Autoversicherung"
"4416","Taxes on cars","4416","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Belastingen personenauto's","Steuern auf Autos"
"4418","Private car use","4418","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Privégebruik personenauto's","Private Pkw-Nutzung"
"4419","Other costs cars","4419","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Overige kosten personenauto's","Sonstige Kosten Autos"
"4421","Fuel other means of transport","4421","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Brandstof overige vervoermiddelen","Kraftstoff für andere Verkehrsmittel"
"4422","Maintenance of other means of transport","4422","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Onderhoud overige vervoermiddelen","Wartung anderer Verkehrsmittel"
"4423","Rental of other means of transport","4423","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Huur overige vervoermiddelen","Anmietung von sonstigen Verkehrsmitteln"
"4424","Lease of other means of transport (operational lease)","4424","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Lease overige vervoermiddelen (operational lease)","Leasing von sonstigen Verkehrsmitteln (Operating Leasing)"
"4425","Insurance other means of transport","4425","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Assurantie overige vervoermiddelen","Versicherung anderer Verkehrsmittel"
"4426","Taxes on other means of transport","4426","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Belastingen overige vervoermiddelen","Steuern auf andere Verkehrsmittel"
"4428","Other remaining means of transport","4428","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Overige overige vervoermiddelen","Andere verbleibende Verkehrsmittel"
"4490","Other transport costs","4490","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_6","False","Overige vervoerskosten","Sonstige Transportkosten"
"4505","Business gifts","4505","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_10","False","Relatiegeschenken","Werbegeschenke"
"4510","Travel and accommodation expenses","4510","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_10","False","Reis- en verblijfkosten","Reise- und Unterbringungskosten"
"4515","Representation costs","4515","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_10","False","Representatiekosten","Repräsentationskosten"
"4520","Advertising costs","4520","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_10","False","Etalagekosten","Kosten für Werbung"
"4525","Advertisements","4525","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_10","False","Advertenties","Inserate"
"4530","Scholarship costs","4530","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_10","False","Beurskosten","Kosten für das Stipendium"
"4535","Advertising costs","4535","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_10","False","Etalagekosten","Kosten für Werbung"
"4540","Website costs","4540","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_10","False","Websitekosten","Kosten der Website"
"4550","Bad debt write-off","4550","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_10","False","Afschrijving dubieuze debiteuren","Abschreibung uneinbringlicher Forderungen"
"4560","Credit limitation debtors","4560","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_10","False","Kredietbeperking debiteuren","Kreditbeschränkung Schuldner"
"4570","Freight costs","4570","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_26","False","Vrachtkosten","Frachtkosten"
"4590","Other selling expenses","4590","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_10","False","Overige verkoopkosten","Sonstige Vertriebskosten"
"4601","Audit fees","4601","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_8","False","Accountantskosten","Prüfungsgebühren"
"4605","Consultancy fees","4605","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_8","False","Advieskosten","Beratungsgebühren"
"4610","Legal costs","4610","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_8","False","Juridische kosten","Gerichtskosten"
"4620","Insurance","4620","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_8","False","Assuranties","Versicherung"
"4621","Administration fee","4621","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_8","False","Administratiekosten","Administration fee"
"4630","Bank charges","4630","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_8","False","Bankkosten","Bankgebühren"
"4690","Other overheads","4690","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_8","False","Overige algemene kosten","Sonstige Gemeinkosten"
"4705","Mortgage interest","4705","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_31","False","Rente hypotheek","Mortgage interest"
"4710","Loan interest","4710","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_31","False","Rente lening","Loan interest"
"4720","Bank current account interest","4720","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_31","False","Rente rekening-courant bank","Bank current account interest"
"4790","Other interest expenses","4790","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_31","False","Overige rentelasten","Other interest expenses"
"4795","Other interest income","4795","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_30","False","Overige rentebaten","Sonstige Zinserträge"
"4805","Goodwill","4805","expense_depreciation","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_22","False","Goodwill","Goodwill"
"4810","Buildings / conversions","4810","expense_depreciation","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_21","False","Gebouwen / verbouwingen","Gebäude/Umbauten"
"4820","Machinery","4820","expense_depreciation","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_21","False","Machines","Maschinenpark"
"4830","Inventory","4830","expense_depreciation","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_21","False","Inventaris","Bestandsaufnahme"
"4840","Transport resources","4840","expense_depreciation","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_21","False","Vervoermiddelen","Transportmittel"
"4901","Book profit fixed assets","4901","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_reso","False","Boekwinst vaste activa","Buchgewinn Anlagevermögen"
"4902","Book loss fixed assets","4902","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_reso","False","Boekverlies vaste activa","Buchverlust Anlagevermögen"
"4910","Small business scheme Turnover tax","4910","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_33","False","Kleine ondernemingsregeling Omzetbelasting","Regelung für Kleinunternehmen Umsatzsteuern"
"4920","Exchange rate differences","4920","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_31","False","Koersverschillen","Wechselkursdifferenzen"
"4930","Payment differences","4930","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_31","False","Betalingsverschillen","Unterschiede bei den Zahlungen"
"4940","Cash differences","4940","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_31","False","Kasverschillen","Differenzbeträge"
"4950","Other income","4950","income_other","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_31","False","Overige baten","Sonstige Einnahmen"
"4960","Other expenses","4960","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_31","False","Overige lasten","Sonstige Ausgaben"
"4970","Other income previous years","4970","income_other","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_31","False","Overige baten voorgaande jaren","Sonstige Einnahmen Vorjahre"
"4980","Other expenses prior years","4980","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_31","False","Overige lasten voorgaande jaren","Sonstige Ausgaben Vorjahre"
"7001","Cost price NL trade goods 1","7001","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Kostprijs NL handelsgoederen 1","Selbstkostenpreis NL Handelswaren 1"
"7002","Cost price within EU trade goods 1","7002","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Kostprijs binnen EU handelsgoederen 1","Selbstkostenpreis innerhalb der EU Handelswaren 1"
"7003","Cost price outside EU trade goods 1","7003","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Kostprijs buiten EU handelsgoederen 1","Selbstkostenpreis außerhalb der EU Handelswaren 1"
"7004","Cost Intercompany trade goods 1","7004","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Kostprijs Intercompany handelsgoederen 1","Kosten konzerninterne Handelswaren 1"
"7005","Cost price NL trade goods 2","7005","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Kostprijs NL handelsgoederen 2","Selbstkostenpreis NL Handelswaren 2"
"7006","Cost price within EU trade goods 2","7006","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Kostprijs binnen EU handelsgoederen 2","Selbstkostenpreis innerhalb der EU Handelswaren 2"
"7007","Cost price outside EU trade goods 2","7007","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Kostprijs buiten EU handelsgoederen 2","Selbstkostenpreis außerhalb der EU Handelswaren 2"
"7008","Cost Intercompany trade goods 2","7008","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Kostprijs Intercompany handelsgoederen 2","Kosten konzerninterne Handelswaren 2"
"7009","Cost price NL trade goods 3","7009","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Kostprijs NL handelsgoederen 3","Selbstkostenpreis NL Handelswaren 3"
"7010","Cost price within EU trade goods 3","7010","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Kostprijs binnen EU handelsgoederen 3","Selbstkostenpreis innerhalb der EU Handelswaren 3"
"7011","Cost price outside EU trade goods 3","7011","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Kostprijs buiten EU handelsgoederen 3","Selbstkostenpreis außerhalb der EU Handelswaren 3"
"7012","Cost Intercompany trade goods 3","7012","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Kostprijs Intercompany handelsgoederen 3","Kosten konzerninterne Handelswaren 3"
"7020","Direct salary costs","7020","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Directe loonkosten","Direkte Lohnkosten"
"7030","Production results","7030","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Productieresultaten","Ergebnisse der Produktion"
"7040","Cost of third-party work","7040","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Kostprijs werk derden","Cost of third-party work"
"7050","Provision for raw material obsolescence","7050","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Voorziening incourante voorraad grondstof","Provision for raw material obsolescence"
"7051","Provision for obsolete supplies of consumables","7051","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Voorziening incourante voorraad hulpstof","Rückstellung für veraltete Verbrauchsgüter"
"7052","Provision for obsolete stock of trade goods","7052","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Voorziening incourante voorraad handelsgoederen","Rückstellung für veraltete Bestände an Handelswaren"
"7060","Purchasing costs","7060","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Inkoopkosten","Purchasing costs"
"7061","Purchasing commission","7061","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Inkoopprovisie","Purchasing commission"
"7062","Freight costs","7062","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Vrachtkosten","Frachtkosten"
"7063","Import costs","7063","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Invoerkosten","Kosten der Einfuhr"
"7064","Purchase bonuses","7064","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Inkoopbonussen","Kaufprämien"
"7065","Payment discount creditors","7065","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Betalingskorting crediteuren","Skontogläubiger"
"7070","Warranty costs","7070","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Garantiekosten","Gewährleistungskosten"
"7071","Change in warranty liability","7071","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Mutatie garantieverplichting","Veränderung der Garantieverpflichtung"
"7080","Price differences cost","7080","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Prijsverschillen kostprijs","Preisunterschiede Kosten"
"7090","Stock adjustment","7090","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Voorraadaanpassing","Anpassung der Bestände"
"7091","Rejected stock","7091","expense_direct_cost","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_7","False","Afgekeurde voorraad","Zurückgewiesene Bestände"
"8001","Turnover NL trade goods 1","8001","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet NL handelsgoederen 1","Umsatz NL Handelswaren 1"
"8002","Turnover within EU trade goods 1","8002","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet binnen EU handelsgoederen 1","Umsatz im EU-Warenhandel 1"
"8003","Turnover outside EU trade goods 1","8003","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet buiten EU handelsgoederen 1","Umsatz außerhalb der EU Handelswaren 1"
"8004","Turnover Intercompany trade goods 1","8004","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet Intercompany handelsgoederen 1","Umsatz konzerninterne Handelswaren 1"
"8005","Turnover NL services 1","8005","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet NL diensten 1","Umsatz NL Dienstleistungen 1"
"8011","Turnover NL trade goods 2","8011","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet NL handelsgoederen 2","Umsatz NL Handelswaren 2"
"8012","Turnover within EU trade goods 2","8012","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet binnen EU handelsgoederen 2","Umsatz im EU-Warenhandel 2"
"8013","Turnover outside EU trade goods 2","8013","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet buiten EU handelsgoederen 2","Umsatz außerhalb der EU Handelswaren 2"
"8014","Turnover Intercompany trade goods 2","8014","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet Intercompany handelsgoederen 2","Umsatz konzerninterne Handelswaren 2"
"8015","Turnover NL services 2","8015","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet NL diensten 2","Umsatz NL Dienstleistungen 2"
"8021","Turnover NL trade goods 3","8021","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet NL handelsgoederen 3","Umsatz NL Handelswaren 3"
"8022","Turnover within EU trade goods 3","8022","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet binnen EU handelsgoederen 3","Umsatz im EU-Warenhandel 3"
"8023","Turnover outside EU trade goods 3","8023","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet buiten EU handelsgoederen 2","Umsatz außerhalb der EU Handelswaren 3"
"8024","Turnover Intercompany trade goods 3","8024","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet Intercompany handelsgoederen 3","Umsatz konzerninterne Handelswaren 3"
"8025","Turnover NL services 3","8025","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet NL diensten 3","Umsatz NL Dienstleistungen 3"
"8041","Turnover NL third-party work","8041","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet NL werk derden","Umsatz NL Fremdarbeiten"
"8042","Turnover within EU third-party work","8042","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet binnen EU werk derden","Umsatz innerhalb der EU bei Arbeiten für Dritte"
"8043","Turnover outside EU third-party work","8043","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet buiten EU werk derden","Umsatz außerhalb der EU Arbeiten für Dritte"
"8044","Turnover Intercompany work third parties","8044","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Omzet Intercompany werk derden","Umsatz konzerninterne Arbeiten Dritte"
"8065","Payment discount debtors","8065","income_other","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Betalingskorting debiteuren","Skonto Debitoren"
"8090","Sundry other income","8090","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Diverse overige opbrengsten","Übrige sonstige Erträge"
"8110","Installation/remote sales within EU trade goods 1","80011","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Installatie/afstandsverkopen binnen EU handelsgoederen 1","Installation/Fernabsatz innerhalb der EU Handelswaren 1"
"8120","Installation/remote sales within EU trade goods 2","80012","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Installatie/afstandsverkopen binnen EU handelsgoederen 2","Installation/Fernabsatz innerhalb der EU Handelswaren 2"
"8130","Installation/remote sales within EU trade goods 3","80013","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Installatie/afstandsverkopen binnen EU handelsgoederen 3","Installation/Fernabsatz innerhalb der EU Handelswaren 3"
"8920","Exchange rate differences","8920","income","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_net","False","Koersverschillen","Wechselkursdifferenzen"
"9011","Result of participating interests 1","9011","income_other","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_16","False","Resultaat deelneming 1","Ergebnis der Beteiligungen 1"
"9012","Result of participating interests 2","9012","income_other","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_16","False","Resultaat deelneming 2","Ergebnis der Beteiligungen 2"
"9013","Result of participating interests 3","9013","income_other","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_16","False","Resultaat deelneming 3","Ergebnis der Beteiligungen 3"
"9150","Reorganisation costs","9150","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_8","False","Reorganisatiekosten","Kosten der Umstrukturierung"
"9200","Error account","9200","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_29","False","Foutenrekening","Fehler Konto"
"9900","Corporate tax","9900","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_33","False","Vennootschapsbelasting","Körperschaftssteuer"
"9950","Corporation tax previous years","9950","expense","l10n_nl.account_tag_prev_years_earnings,l10n_nl.account_tag_33","False","Vennootschapsbelasting voorgaande jaren","Corporation tax previous years"

```

## File: data\template\account.fiscal.position-nl.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id","name@nl","name@de"
"fiscal_position_template_national","1","Domestic","1","1","base.nl","","","","","","Binnenland","Inländische"
"fiscal_position_template_transferred","","VAT reverse charge","","","","","btw_21_buy","btw_ink_0","","","BTW verlegd","Umkehrung der Steuerschuldnerschaft"
"fiscal_position_template_eu_private","2","EU private B2C","1","","","base.europe","","","","","",""
"fiscal_position_template_eu","3","EU intra B2B","1","1","","base.europe","btw_0","btw_X0_producten","","","",""
"","","","","","","","btw_9","btw_X0_producten","","","",""
"","","","","","","","btw_21","btw_X0_producten","","","",""
"","","","","","","","btw_overig","btw_X0_producten","","","",""
"","","","","","","","btw_0_d","btw_X0_diensten","","","",""
"","","","","","","","btw_9_d","btw_X0_diensten","","","",""
"","","","","","","","btw_21_d","btw_X0_diensten","","","",""
"","","","","","","","btw_9_buy","btw_I_9","","","",""
"","","","","","","","btw_21_buy","btw_I_21","","","",""
"","","","","","","","btw_9_buy_d","btw_I_9_d","","","",""
"","","","","","","","btw_21_buy_d","btw_I_21_d","","","",""
"","","","","","","","","","7001","7002",""
"","","","","","","","","","7005","7006",""
"","","","","","","","","","7009","7010",""
"","","","","","","","","","8001","8002",""
"","","","","","","","","","8011","8012",""
"","","","","","","","","","8021","8022",""
"fiscal_position_template_non_eu","4","EU non-intra","1","","","","btw_0","btw_X1","","","Niet-EU landen","EU nicht-intra"
"","","","","","","","btw_9_buy","btw_E1_9","","","",""
"","","","","","","","btw_21","btw_X1","","","",""
"","","","","","","","btw_0_d","btw_X1","","","",""
"","","","","","","","btw_9_buy_d","btw_E1_9","","","",""
"","","","","","","","btw_21_d","btw_X1","","","",""
"","","","","","","","btw_21_buy","btw_E2","","","",""
"","","","","","","","btw_21_buy_d","btw_E2","","","",""
"","","","","","","","","","7001","7003",""
"","","","","","","","","","7005","7007",""
"","","","","","","","","","7009","7011",""
"","","","","","","","","","8001","8003",""
"","","","","","","","","","8011","8013",""
"","","","","","","","","","8021","8023",""
"fiscal_position_template_eu_no_taxes_report","","Installation and Distance Selling","","","","","btw_0","btw_X2","","","Installatie en Afstandsverkopen","Installation und Fernabsatz"
"","","","","","","","btw_verk_0","btw_X2","","","",""
"","","","","","","","btw_9","btw_X2","","","",""
"","","","","","","","btw_21","btw_X2","","","",""
"","","","","","","","btw_overig","btw_X2","","","",""
"","","","","","","","btw_0_d","btw_X2","","","",""
"","","","","","","","btw_9_d","btw_X2","","","",""
"","","","","","","","btw_21_d","btw_X2","","","",""
"","","","","","","","btw_overig_d","btw_X2","","","",""
"","","","","","","","","","8001","8110",""
"","","","","","","","","","8011","8120",""
"","","","","","","","","","8021","8130",""

```

## File: data\template\account.tax-nl.csv

```csv
"id","sequence","name","description","invoice_label","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@nl","invoice_label@nl","description@de","invoice_label@de"
"btw_0","10","0%","Sales/turnover untaxed (zero rate)","0% VAT","0.0","percent","sale","tax_group_0","base","invoice","+1e (omzet)","","","Verkopen/omzet onbelast (nul-tarief) diensten","0% BTW","Verkauf/Umsatz unversteuerte (Nullsatz) Dienstleistungen","0% MwSt"
"","","","","","","","","","tax","invoice","","","","","","",""
"","","","","","","","","","base","refund","-1e (omzet)","","","","","",""
"","","","","","","","","","tax","refund","","","","","","",""
"btw_9","10","9% ST","Sales/turnover low 9%","9% VAT","9.0","percent","sale","tax_group_9","base","invoice","+1b (omzet)","","","BTW te vorderen laag (inkopen) 9%","9% BTW","Niedrige Mehrwertsteuerforderungen (Käufe) 9%","9% MwSt"
"","","","","","","","","","tax","invoice","+1b (BTW)","vat_payable_l","","","","",""
"","","","","","","","","","base","refund","-1b (omzet)","","","","","",""
"","","","","","","","","","tax","refund","-1b (BTW)","vat_payable_l","","","","",""
"btw_21","5","21% ST","Sales/turnover high","21% VAT","21.0","percent","sale","tax_group_21","base","invoice","+1a (omzet)","","","Verkopen/omzet hoog","21% BTW","Absatz/Umsatz hoch","21% MwSt"
"","","","","","","","","","tax","invoice","+1a (BTW)","vat_payable_h","","","","",""
"","","","","","","","","","base","refund","-1a (omzet)","","","","","",""
"","","","","","","","","","tax","refund","-1a (BTW)","vat_payable_h","","","","",""
"btw_overig","15","21% ST O","Sales/turnover other","variable VAT","21.0","percent","sale","tax_group_21","base","invoice","+1a (omzet)","","","Verkopen/omzet overig diensten","variabel BTW","Absatz/Umsatz sonstige Dienstleistungen","variabel MwSt"
"","","","","","","","","","tax","invoice","+1a (BTW)","vat_payable_h","","","","",""
"","","","","","","","","","base","refund","-1c (omzet)","","","","","",""
"","","","","","","","","","tax","refund","-1c (BTW)","vat_payable_h","","","","",""
"btw_0_d","10","0% ST S","Sales/turnover untaxed (zero rate) services","0% VAT","0.0","percent","sale","tax_group_0","base","invoice","+1e (omzet)","","","Verkopen/omzet onbelast (nul-tarief) diensten","0% BTW","Verkauf/Umsatz unversteuerte (Nullsatz) Dienstleistungen","0% MwSt"
"","","","","","","","","","tax","invoice","","","","","","",""
"","","","","","","","","","base","refund","-1e (omzet)","","","","","",""
"","","","","","","","","","tax","refund","","","","","","",""
"btw_9_d","10","9% ST S","Sales/turnover low services 9%","9% VAT","9.0","percent","sale","tax_group_9","base","invoice","+1b (omzet)","","","Verkopen/omzet laag diensten 9%","9% BTW","Absatz/Umsatz geringe Dienstleistungen 9%","9% MwSt"
"","","","","","","","","","tax","invoice","+1b (BTW)","vat_payable_l_d","","","","",""
"","","","","","","","","","base","refund","-1b (omzet)","","","","","",""
"","","","","","","","","","tax","refund","-1b (BTW)","vat_payable_l_d","","","","",""
"btw_21_d","6","21% ST S","Sales/turnover high services","21% VAT","21.0","percent","sale","tax_group_21","base","invoice","+1a (omzet)","","","Verkopen/omzet hoog diensten","21% BTW","Absatz/Umsatz hohe Dienstleistungen","21% MwSt"
"","","","","","","","","","tax","invoice","+1a (BTW)","vat_payable_h_d","","","","",""
"","","","","","","","","","base","refund","-1a (omzet)","","","","","",""
"","","","","","","","","","tax","refund","-1a (BTW)","vat_payable_h_d","","","","",""
"btw_overig_d","15","21% ST S O","Sales/turnover other services","variable VAT services","21.0","percent","sale","tax_group_21","base","invoice","+1a (omzet)","","","Verkopen/omzet overig diensten","variabel BTW diensten","Absatz/Umsatz sonstige Dienstleistungen","variable MwSt.-Dienstleistungen"
"","","","","","","","","","tax","invoice","+1a (BTW)","vat_payable_h_d","","","","",""
"","","","","","","","","","base","refund","-1c (omzet)","","","","","",""
"","","","","","","","","","tax","refund","-1c (BTW)","vat_payable_h_d","","","","",""
"btw_9_buy","10","9%","VAT receivable low (purchases) 9%","9% VAT","9.0","percent","purchase","tax_group_9","base","invoice","","","","BTW te vorderen laag (inkopen) 9%","9% BTW","Niedrige Mehrwertsteuerforderungen (Käufe) 9%","9% MwSt"
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_l","","","","",""
"","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_l","","","","",""
"btw_21_buy","5","21%","VAT receivable high (purchases)","21% VAT","21.0","percent","purchase","tax_group_21","base","invoice","","","","BTW te vorderen hoog (inkopen)","21% BTW","Hohe Mehrwertsteuerforderungen (Käufe)","21% MwSt"
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_h","","","","",""
"","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_h","","","","",""
"btw_overig_buy","15","21% O","VAT receivable other (purchases)","variable VAT","21.0","percent","purchase","tax_group_21","base","invoice","","","","BTW te vorderen overig (inkopen) diensten","variabel BTW","MwSt.-Forderungen Sonstige (Käufe) Dienstleistungen","MwSt variabel"
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_h","","","","",""
"","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_h","","","","",""
"btw_9_buy_d","10","9% S","VAT receivable low (purchases) services 9%","9% VAT","9.0","percent","purchase","tax_group_9","base","invoice","","","","BTW te vorderen laag (inkopen) diensten 9%","9% BTW","MwSt.-Empfänger niedrige (Käufe) Dienstleistungen 9%","21% MwSt"
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_l_d","","","","",""
"","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_l_d","","","","",""
"btw_21_buy_d","6","21% S","VAT receivable high (purchases) services","21% VAT","21.0","percent","purchase","tax_group_21","base","invoice","","","","BTW te vorderen hoog (inkopen) diensten","21% BTW","MwSt.-Empfänger hohe (Käufe) Dienstleistungen","21% MwSt"
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_h_d","","","","",""
"","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_h_d","","","","",""
"btw_overig_buy_d","15","21% S O","VAT receivable other (purchases) services","variable VAT services","21.0","percent","purchase","tax_group_21","base","invoice","","","","BTW te vorderen overig (inkopen) diensten","variabel BTW diensten","MwSt.-Forderungen Sonstige (Käufe) Dienstleistungen","variable MwSt.-Dienstleistungen"
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_h_d","","","","",""
"","","","","","","","","","base","refund","","","","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_h_d","","","","",""
"btw_verk_0","15","0% R","Referred VAT to be paid (sales)","0% VAT reverse charge","0.0","percent","sale","tax_group_0","base","invoice","+1e (omzet)","","","BTW af te dragen verlegd (verkopen)","0% BTW verlegd","Zu zahlende Referenz-Mehrwertsteuer (Umsatz)","0% MwSt verlegd"
"","","","","","","","","","tax","invoice","","","","","","",""
"","","","","","","","","","base","refund","-1e (omzet)","","","","","",""
"","","","","","","","","","tax","refund","","","","","","",""
"btw_ink_0","15","21% R","Referred VAT to be paid (purchases)","21% VAT reverse charge","21.0","percent","purchase","tax_group_21","base","invoice","+2a (omzet)","","","BTW af te dragen verlegd (inkopen)","21% BTW verlegd","Zu entrichtende Mehrwertsteuer (Käufe)","21% MwSt. übertragen"
"","","","","","","","","","tax","invoice","+2a (BTW)","vat_payable_v","","","","",""
"","","","","","","","","","tax","invoice","-5b (BTW)","vat_payable_v","-100","","","",""
"","","","","","","","","","base","refund","-2a (omzet)","","","","","",""
"","","","","","","","","","tax","refund","-2a (BTW)","vat_payable_v","","","","",""
"","","","","","","","","","tax","refund","+5b (BTW)","vat_payable_v","-100","","","",""
"btw_I_9","20","9% EX EU","Imports within EU 9%","0% EU","9.0","percent","purchase","tax_group_9","base","invoice","+4b (omzet)","","","Inkopen import binnen EU laag diensten 9%","0% EU","Käufe von Einfuhren innerhalb der EU geringe Dienstleistungen 9%","0% EU"
"","","","","","","","","","tax","invoice","-4b (BTW)","vat_payable_l_eu","-100","","","",""
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_l_eu","","","","",""
"","","","","","","","","","base","refund","-4b (omzet)","","","","","",""
"","","","","","","","","","tax","refund","+4b (BTW)","vat_payable_l_eu","-100","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_l_eu","","","","",""
"btw_I_21","20","21% EX EU","Purchases of imports within EU high","21% EU","21.0","percent","purchase","tax_group_21","base","invoice","+4b (omzet)","","","Inkopen import binnen EU hoog","0% EU","Käufe von Einfuhren innerhalb der EU hoch","0% EU"
"","","","","","","","","","tax","invoice","-4b (BTW)","vat_payable_h_eu","-100","","","",""
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_h_eu","","","","",""
"","","","","","","","","","base","refund","-4b (omzet)","","","","","",""
"","","","","","","","","","tax","refund","+4b (BTW)","vat_payable_h_eu","-100","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_h_eu","","","","",""
"btw_X0_producten","20","0% EX EU G","Export sales within EU (products)","0% EU","0.0","percent","sale","tax_group_0","base","invoice","+3bg (omzet)","","","Verkopen export binnen EU (producten)","0% EU","Exportverkäufe innerhalb der EU (Produkte)","0% EU"
"","","","","","","","","","tax","invoice","","","","","","",""
"","","","","","","","","","base","refund","-3bg (omzet)","","","","","",""
"","","","","","","","","","tax","refund","","","","","","",""
"btw_X0_ABC_levering","20","0% EX EU T","A-B-C supply chain transactions within the EU","0% EU","0.0","percent","sale","tax_group_0","base","invoice","+3bt (omzet)","","","ABC-levering binnen EU","0% EU","ABC-Lieferung innerhalb der EU","ABC-Lieferung innerhalb der EU"
"","","","","","","","","","tax","invoice","","","","","","",""
"","","","","","","","","","base","refund","-3bt (omzet)","","","","","",""
"","","","","","","","","","tax","refund","","","","","","",""
"btw_X0_diensten","20","0% EX EU S","Export sales within EU (services)","0% EU","0.0","percent","sale","tax_group_0","base","invoice","+3bs (omzet)","","","Verkopen export binnen EU (diensten)","0% EU","Exportverkäufe innerhalb der EU (Dienstleistungen)","0% EU"
"","","","","","","","","","tax","invoice","","","","","","",""
"","","","","","","","","","base","refund","-3bs (omzet)","","","","","",""
"","","","","","","","","","tax","refund","","","","","","",""
"btw_X2","20","0% EU I","Installation/remote sales within EU","Inst./rem. sales within EU","0.0","percent","sale","tax_group_0","base","invoice","+3c (omzet)","","","Installatie/afstandsverkopen binnen EU","Inst./afst.verkopen binnen EU","Installation/Fernabsatz innerhalb der EU","Inst./Verkäufe innerhalb der EU"
"","","","","","","","","","tax","invoice","","","","","","",""
"","","","","","","","","","base","refund","-3c (omzet)","","","","","",""
"","","","","","","","","","tax","refund","","","","","","",""
"btw_I_9_d","20","9% EX EU S","Purchasing imports within EU low services 9%","0% EU","9.0","percent","purchase","tax_group_9","base","invoice","+4b (omzet)","","","Inkopen import binnen EU laag diensten 9%","0% EU","Käufe von Einfuhren innerhalb der EU geringe Dienstleistungen 9%","0% EU"
"","","","","","","","","","tax","invoice","-4b (BTW)","vat_payable_l_d_eu","-100","","","",""
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_l_d_eu","","","","",""
"","","","","","","","","","base","refund","-4b (omzet)","","","","","",""
"","","","","","","","","","tax","refund","+4b (BTW)","vat_payable_l_d_eu","-100","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_l_d_eu","","","","",""
"btw_I_21_d","20","21% EX EU S","Purchasing imports within EU high services","0% EU","21.0","percent","purchase","tax_group_21","base","invoice","+4b (omzet)","","","Inkopen import binnen EU hoog diensten","0% EU","Einkauf von Importen innerhalb der EU - hohe Dienstleistungen",""
"","","","","","","","","","tax","invoice","-4b (BTW)","vat_payable_h_d_eu","-100","","","",""
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_h_d_eu","","","","",""
"","","","","","","","","","base","refund","-4b (omzet)","","","","","",""
"","","","","","","","","","tax","refund","+4b (BTW)","vat_payable_h_d_eu","-100","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_h_d_eu","","","","",""
"btw_E1_9","20","9% EX O EU","Buying imports outside EU low 9%","0% Non-EU","9.0","percent","purchase","tax_group_9","base","invoice","+4a (omzet)","","","Inkopen import buiten EU laag 9%","0% Buiten-EU","Kauf von Einfuhren außerhalb der EU niedrig 9%","0% Außerhalb der EU"
"","","","","","","","","","tax","invoice","-4a (BTW)","vat_payable_l_non_eu","-100","","","",""
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_l_non_eu","","","","",""
"","","","","","","","","","base","refund","-4a (omzet)","","","","","",""
"","","","","","","","","","tax","refund","+4a (BTW)","vat_payable_l_non_eu","-100","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_l_non_eu","","","","",""
"btw_E2","20","21% EX O EU","Buying imports outside EU high","0% Non-EU","21.0","percent","purchase","tax_group_21","base","invoice","+4a (omzet)","","","Inkopen import buiten EU hoog","0% Buiten-EU","Käufe von Einfuhren außerhalb der EU hoch","0% Außerhalb der EU"
"","","","","","","","","","tax","invoice","-4a (BTW)","vat_payable_h_non_eu","-100","","","",""
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_h_non_eu","","","","",""
"","","","","","","","","","base","refund","-4a (omzet)","","","","","",""
"","","","","","","","","","tax","refund","+4a (BTW)","vat_payable_h_non_eu","-100","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_h_non_eu","","","","",""
"btw_E_overig","20","21% EX O EU O","VAT import outside EU other purchases","VAT import outside EU other purchases","21.0","percent","purchase","tax_group_21","base","invoice","+4a (omzet)","","","Inkopen import buiten EU overig","BTW import buiten EU overig inkopen","Einfuhren außerhalb der EU andere","MwSt. Einfuhr außerhalb der EU Sonstige Käufe"
"","","","","","","","","","tax","invoice","-4a (BTW)","vat_payable_h_non_eu","-100","","","",""
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_h_non_eu","","","","",""
"","","","","","","","","","base","refund","-4a (omzet)","","","","","",""
"","","","","","","","","","tax","refund","+4a (BTW)","vat_payable_h_non_eu","-100","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_h_non_eu","","","","",""
"btw_X1","20","0% EX","Export sales outside EU","VAT export outside EU","0.0","percent","sale","tax_group_0","base","invoice","+3a (omzet)","","","Verkopen export buiten EU","BTW export buiten EU","Exportverkäufe außerhalb der EU","Mehrwertsteuerausfuhr außerhalb der EU"
"","","","","","","","","","tax","invoice","","","","","","",""
"","","","","","","","","","base","refund","-3a (omzet)","","","","","",""
"","","","","","","","","","tax","refund","","","","","","",""
"btw_X3","21","0% EX I","Installation/remote sales outside EU","Installation/remote sales outside EU","0.0","percent","sale","tax_group_0","base","invoice","+3a (omzet)","","","Installatie/afstandsverkopen buiten EU","Inst./afst.verkopen buiten EU","Installation/Fernabsatz außerhalb der EU","Inst./Verkäufe außerhalb der EU"
"","","","","","","","","","tax","invoice","","","","","","",""
"","","","","","","","","","base","refund","-3a (omzet)","","","","","",""
"","","","","","","","","","tax","refund","","","","","","",""
"btw_E1_d_9","20","9% EX O EU S","Purchasing imports outside EU low services 9%","0% Non-EU","9.0","percent","purchase","tax_group_9","base","invoice","+4a (omzet)","","","Inkopen import buiten EU laag diensten 9%","0% Buiten-EU","Käufe von Einfuhren außerhalb der EU geringe Dienstleistungen 9%","0% Außerhalb der EU"
"","","","","","","","","","tax","invoice","-4a (BTW)","vat_payable_l_d_non_eu","-100","","","",""
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_l_d_non_eu","","","","",""
"","","","","","","","","","base","refund","-4a (omzet)","","","","","",""
"","","","","","","","","","tax","refund","+4a (BTW)","vat_payable_l_d_non_eu","-100","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_l_d_non_eu","","","","",""
"btw_E2_d","20","21% EX O EU S","Purchasing imports outside EU high services","0% Non-EU","21.0","percent","purchase","tax_group_21","base","invoice","+4a (omzet)","","","Inkopen import buiten EU hoog diensten","0% Buiten-EU","Einkauf von Importen außerhalb der EU hohe Dienstleistungen","0% Außerhalb der EU"
"","","","","","","","","","tax","invoice","-4a (BTW)","vat_payable_h_d_non_eu","-100","","","",""
"","","","","","","","","","tax","invoice","+5b (BTW)","vat_refund_h_d_non_eu","","","","",""
"","","","","","","","","","base","refund","-4a (omzet)","","","","","",""
"","","","","","","","","","tax","refund","+4a (BTW)","vat_payable_h_d_non_eu","-100","","","",""
"","","","","","","","","","tax","refund","-5b (BTW)","vat_refund_h_d_non_eu","","","","",""

```

## File: data\template\account.tax.group-nl.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id","name@nl","name@de"
"tax_group_0","VAT 0%","base.nl","pay","pay","BTW 0%","MWST. 0%"
"tax_group_9","VAT 9%","base.nl","pay","pay","BTW 9%","MWST. 9%"
"tax_group_21","VAT 21%","base.nl","pay","pay","BTW 21%","MWST. 21%"

```

## File: migrations\3.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'nl')], order="parent_path"):
        env['account.chart.template'].try_loading('nl', company, force_create=False)

```

## File: migrations\3.3\post-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID
from odoo.tools import SQL


def _get_tax_ids_for_xml_id(env, xml_id):
    rows = env.execute_query(SQL(
        """
        SELECT res_id
        FROM ir_model_data
        WHERE model = 'account.tax'
        AND name LIKE %s
        """, f"%{xml_id}"
    ))
    return [res_id for res_id, in rows]


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})

    goods_tax_ids = [tax_id for xml_id in ['btw_X0_producten', 'btw_X0']
                     for tax_id in _get_tax_ids_for_xml_id(env, xml_id)]

    goods_taxes = env['account.tax'].browse(goods_tax_ids)
    services_taxes = env['account.tax'].browse(_get_tax_ids_for_xml_id(env, 'btw_X0_diensten'))

    old_3bl_tax_tags = env['account.account.tag']._get_tax_tags('3bl (omzet)', 'nl')
    old_3b_tax_tags = env['account.account.tag']._get_tax_tags('3b (omzet)', 'nl')
    if not old_3bl_tax_tags and not old_3b_tax_tags:
        return

    goods_tax_tags = env['account.account.tag']._get_tax_tags('3bg (omzet)', 'nl')
    services_tax_tags = env['account.account.tag']._get_tax_tags('3bs (omzet)', 'nl')

    old_3bl_tax_tags_plus = old_3bl_tax_tags.filtered(lambda tag: not tag.tax_negate).id
    old_3b_tax_tags_plus = old_3b_tax_tags.filtered(lambda tag: not tag.tax_negate).id
    old_plus_tax_tag_ids = []
    if old_3bl_tax_tags_plus:
        old_plus_tax_tag_ids.append(old_3bl_tax_tags_plus)
    if old_3b_tax_tags_plus:
        old_plus_tax_tag_ids.append(old_3b_tax_tags_plus)

    old_3bl_tax_tags_minus = old_3bl_tax_tags.filtered(lambda tag: tag.tax_negate).id
    old_3b_tax_tags_minus = old_3b_tax_tags.filtered(lambda tag: tag.tax_negate).id
    old_minus_tax_tag_ids = []
    if old_3bl_tax_tags_minus:
        old_minus_tax_tag_ids.append(old_3bl_tax_tags_minus)
    if old_3b_tax_tags_minus:
        old_minus_tax_tag_ids.append(old_3b_tax_tags_minus)

    goods_plus_tax_tag_id = goods_tax_tags.filtered(lambda tag: not tag.tax_negate).id
    goods_minus_tax_tag_id = goods_tax_tags.filtered(lambda tag: tag.tax_negate).id
    services_plus_tax_tag_id = services_tax_tags.filtered(lambda tag: not tag.tax_negate).id
    services_minus_tax_tag_id = services_tax_tags.filtered(lambda tag: tag.tax_negate).id

    insert_query_params = [
        (goods_plus_tax_tag_id, goods_taxes.ids, old_plus_tax_tag_ids, goods_taxes.invoice_repartition_line_ids.ids),
        (services_plus_tax_tag_id, services_taxes.ids, old_plus_tax_tag_ids, services_taxes.invoice_repartition_line_ids.ids),
        (goods_minus_tax_tag_id, goods_taxes.ids, old_minus_tax_tag_ids, goods_taxes.refund_repartition_line_ids.ids),
        (services_minus_tax_tag_id, services_taxes.ids, old_minus_tax_tag_ids, services_taxes.refund_repartition_line_ids.ids),
    ]
    insert_query_parts = []

    for new_tax_tag_id, tax_ids, old_tax_tag_ids, repartition_line_ids in insert_query_params:
        insert_query_parts.append(
            SQL("""
                    SELECT tag_aml_rel.account_move_line_id, %s
                    FROM account_account_tag_account_move_line_rel tag_aml_rel
                    JOIN account_move_line_account_tax_rel aml_at_rel ON aml_at_rel.account_move_line_id = tag_aml_rel.account_move_line_id
                    WHERE aml_at_rel.account_tax_id = ANY(%s)
                    AND tag_aml_rel.account_account_tag_id = ANY(%s)
                    """,
                    new_tax_tag_id, tax_ids, old_tax_tag_ids
            )
        )

        if len(old_tax_tag_ids) > 1:
            cr.execute(SQL(
                """
                DELETE FROM account_account_tag_account_tax_repartition_line_rel tag_aml_rel
                WHERE tag_aml_rel.account_account_tag_id = %s
                AND (
                    SELECT COUNT(*)
                    FROM account_account_tag_account_tax_repartition_line_rel sub_tag_aml_rel
                    WHERE sub_tag_aml_rel.account_tax_repartition_line_id = tag_aml_rel.account_tax_repartition_line_id
                    AND sub_tag_aml_rel.account_account_tag_id = %s
                ) >= 1
                """,
                old_tax_tag_ids[0], old_tax_tag_ids[1]
            ))

        cr.execute(SQL(
            """
            UPDATE account_account_tag_account_tax_repartition_line_rel
            SET account_account_tag_id = %s
            WHERE account_tax_repartition_line_id = ANY(%s)
            AND account_account_tag_id = ANY(%s)
            """,
            new_tax_tag_id, repartition_line_ids, old_tax_tag_ids
        ))

    cr.execute(SQL(
            """
            INSERT INTO account_account_tag_account_move_line_rel (account_move_line_id, account_account_tag_id)
                %s
            ON CONFLICT DO NOTHING
            """,
            SQL(" UNION ").join(insert_query_parts)
    ))

```

## File: migrations\3.4\end-migrate.py

```python
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'nl')], order="parent_path"):
        env['account.chart.template'].try_loading('nl', company, force_create=False)

```

## File: migrations\9.0.2.0\pre-set_tags_and_taxes_updatable.py

```python
# -*- coding: utf-8 -*-

from odoo.modules.registry import Registry

def migrate(cr, version):
    registry = Registry(cr.dbname)
    from odoo.addons.account.models.chart_template import migrate_set_tags_and_taxes_updatable
    migrate_set_tags_and_taxes_updatable(cr, registry, 'l10n_nl')

```

## File: models\account_chart_template.py

```python
# -*- coding: utf-8 -*-

from odoo import models


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    def _post_load_data(self, template_code, company, template_data):
        super()._post_load_data(template_code, company, template_data)
        if template_code == 'nl':
            if cash_tag := self.env.ref('l10n_nl.account_tag_25', raise_if_not_found=False):
                company.account_journal_suspense_account_id.tag_ids += cash_tag
                company.transfer_account_id.tag_ids += cash_tag
            if undist_profit_tag := self.env.ref('l10n_nl.account_tag_undist_profit', raise_if_not_found=False):
                company.get_unaffected_earnings_account().tag_ids += undist_profit_tag

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-

from odoo import api, fields, models, _


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    @api.model
    def _prepare_liquidity_account_vals(self, company, code, vals):
        # OVERRIDE
        account_vals = super()._prepare_liquidity_account_vals(company, code, vals)

        if company.account_fiscal_country_id.code == 'NL':
            # Ensure the newly liquidity accounts have the right account tag in order to be part
            # of the Dutch financial reports.
            account_vals.setdefault('tag_ids', [])
            account_vals['tag_ids'].append((4, self.env.ref('l10n_nl.account_tag_25').id))

        return account_vals

```

## File: models\res_company.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_nl_rounding_difference_loss_account_id = fields.Many2one('account.account', check_company=True)
    l10n_nl_rounding_difference_profit_account_id = fields.Many2one('account.account', check_company=True)

```

## File: models\template_nl.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('nl')
    def _get_nl_template_data(self):
        return {
            'code_digits': '6',
            'property_account_receivable_id': 'recv',
            'property_account_payable_id': 'pay',
            'property_account_expense_categ_id': '7001',
            'property_account_income_categ_id': '8001',
            'property_stock_account_input_categ_id': '1450',
            'property_stock_account_output_categ_id': '1250',
            'property_stock_valuation_account_id': '3200',
        }

    @template('nl', 'res.company')
    def _get_nl_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.nl',
                'bank_account_code_prefix': '103',
                'cash_account_code_prefix': '101',
                'transfer_account_code_prefix': '1060',
                'account_default_pos_receivable_account_id': 'recv_pos',
                'income_currency_exchange_account_id': '8920',
                'expense_currency_exchange_account_id': '4920',
                'account_journal_early_pay_discount_loss_account_id': '7065',
                'account_journal_early_pay_discount_gain_account_id': '8065',
                'l10n_nl_rounding_difference_loss_account_id': '4960',
                'l10n_nl_rounding_difference_profit_account_id': '4950',
                'account_sale_tax_id': 'btw_21',
                'account_purchase_tax_id': 'btw_21_buy',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_nl
from . import account_journal
from . import account_chart_template
from . import res_company

```

## File: views\res_config_settings_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record model="ir.ui.view" id="res_config_settings_view_form">
        <field name="name">res.config.settings.view.form</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//app[@name='account']/block" position="after">
                <div id="dutch_localization_section" invisible="1">
                    <block title="Dutch Localization" id="dutch_localization" invisible="country_code != 'NL'">
                    </block>
                </div>
            </xpath>
        </field>
    </record>

</odoo>

```

