# Odoo Module: l10n_ca

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2010 Savoir-faire Linux (<https://www.savoirfairelinux.com>).

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Canada - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['ca'],
    'author': 'Savoir-faire Linux (https://www.savoirfairelinux.com); Odoo SA',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the module to manage the Canadian accounting chart in Odoo.
===========================================================================================

Canadian accounting charts and localizations.

Fiscal positions
----------------

When considering taxes to be applied, it is the province where the delivery occurs that matters.
Therefore we decided to implement the most common case in the fiscal positions: delivery is the
responsibility of the vendor and done at the customer location.

Some examples:

1) You have a customer from another province and you deliver to his location.
On the customer, set the fiscal position to his province.

2) You have a customer from another province. However this customer comes to your location
with their truck to pick up products. On the customer, do not set any fiscal position.

3) An international vendor doesn't charge you any tax. Taxes are charged at customs
by the customs broker. On the vendor, set the fiscal position to International.

4) An international vendor charge you your provincial tax. They are registered with your
position.
    """,
    'depends': [
        'account',
        'base_iban',
    ],
    'auto_install': ['account'],
    'data': [
        'data/tax_report.xml',
        'views/res_partner_view.xml',
        'views/res_company_view.xml',
        'views/report_invoice.xml',
        'views/report_template.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\tax_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="l10n_ca_tr_gsthst" model="account.report">
        <field name="name">GST/HST report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ca"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_ca_tr_gsthst_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_ca_tr_gsthst_90" model="account.report.line">
                <field name="name">90 - Taxable sales including zero-rated supplies (other than zero‑rated exports) made in Canada</field>
                <field name="code">Ca90</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_gsthst_90_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Ca90</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_gsthst_91" model="account.report.line">
                <field name="name">91 - Exempt supplies and zero-rated exports</field>
                <field name="code">Ca91</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_gsthst_91_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Ca91</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_gsthst_101" model="account.report.line">
                <field name="name">101 - Sales and other revenues</field>
                <field name="code">Ca101</field>
                <field name="hierarchy_level">1</field>
                <field name="aggregation_formula">Ca90.balance + Ca91.balance</field>
            </record>
            <record id="l10n_ca_tr_gsthst_103" model="account.report.line">
                <field name="name">103 - GST and HST amounts collected or that became collectible</field>
                <field name="code">Ca103</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_gsthst_103_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Ca103</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_gsthst_104" model="account.report.line">
                <field name="name">104 - Adjustments to be added to the net tax for the reporting period (for example, the GST/HST obtained from the recovery of a bad debt).</field>
                <field name="code">Ca104</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_gsthst_104_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Ca104</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_gsthst_105" model="account.report.line">
                <field name="name">105 - GST/HST and adjustments (103 + 104)</field>
                <field name="code">Ca105</field>
                <field name="aggregation_formula">Ca103.balance + Ca104.balance</field>
            </record>
            <record id="l10n_ca_tr_gsthst_106" model="account.report.line">
                <field name="name">106 - GST/HST you paid or that is payable by you on qualifying expenses (input tax credits – ITCs) for the current period and any eligible unclaimed ITCs from a previous period</field>
                <field name="code">Ca106</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_gsthst_106_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Ca106</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_gsthst_107" model="account.report.line">
                <field name="name">107 - Adjustments to be deducted when determining the net tax for the reporting period (for example, the GST/HST included in a bad debt)</field>
                <field name="code">Ca107</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_gsthst_107_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Ca107</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_gsthst_108" model="account.report.line">
                <field name="name">108 - ITCs and adjustments</field>
                <field name="code">Ca108</field>
                <field name="aggregation_formula">Ca106.balance + Ca107.balance</field>
            </record>
            <record id="l10n_ca_tr_gsthst_109" model="account.report.line">
                <field name="name">109 - Net tax</field>
                <field name="code">Ca109</field>
                <field name="aggregation_formula">Ca105.balance - Ca108.balance</field>
            </record>
            <record id="l10n_ca_tr_gsthst_110" model="account.report.line">
                <field name="name">110 - Instalment and other annual filer payments</field>
                <field name="code">Ca110</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_gsthst_110_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Ca110</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_gsthst_111" model="account.report.line">
                <field name="name">111 - GST/HST rebates claimable</field>
                <field name="code">Ca111</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_gsthst_111_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Ca111</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_gsthst_112" model="account.report.line">
                <field name="name">112 - Total other credits (if applicable)</field>
                <field name="code">Ca112</field>
                <field name="aggregation_formula">Ca110.balance + Ca111.balance</field>
            </record>
            <record id="l10n_ca_tr_gsthst_113A" model="account.report.line">
                <field name="name">113A - Balance after credits</field>
                <field name="code">Ca113A</field>
                <field name="aggregation_formula">Ca109.balance - Ca112.balance</field>
            </record>
            <record id="l10n_ca_tr_gsthst_205" model="account.report.line">
                <field name="name">205 - GST/HST due on purchases of real property or purchases of emission allowances</field>
                <field name="code">Ca205</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_gsthst_205_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Ca205</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_gsthst_405" model="account.report.line">
                <field name="name">405 - GST/HST to be self-assessed</field>
                <field name="code">Ca405</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_gsthst_405_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">Ca405</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_gsthst_113B" model="account.report.line">
                <field name="name">113B - Total other debits (if applicable)</field>
                <field name="code">Ca113B</field>
                <field name="aggregation_formula">Ca205.balance + Ca405.balance</field>
            </record>
            <record id="l10n_ca_tr_gsthst_113C" model="account.report.line">
                <field name="name">113C - Balance after debits</field>
                <field name="code">Ca113C</field>
                <field name="aggregation_formula">Ca113A.balance - Ca113B.balance</field>
            </record>
            <record id="l10n_ca_tr_gsthst_114" model="account.report.line">
                <field name="name">114 - Refund Claimable</field>
                <field name="code">Ca114</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_gsthst_114_aggr" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">-Ca113C.balance</field>
                        <field name="subformula">if_above(CAD(0))</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_gsthst_115" model="account.report.line">
                <field name="name">115 - Amount to pay</field>
                <field name="code">Ca115</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_gsthst_115_aggr" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">Ca113C.balance</field>
                        <field name="subformula">if_above(CAD(0))</field>
                    </record>
                </field>
            </record>
        </field>
    </record>

    <record id="l10n_ca_tr_pst_bc" model="account.report">
        <field name="name">British-Columbia PST report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ca"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_ca_tr_pst_bc_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_ca_tr_pst_bc_a" model="account.report.line">
                <field name="name">A - Sales and Leases (excluding GST/PST)</field>
                <field name="code">CaBcA</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_pst_bc_a_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaBcA</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_pst_bc_B" model="account.report.line">
                <field name="name">B - PST Collectable on Sales and Leases</field>
                <field name="code">CaBcB</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_pst_bc_b_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaBcB</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_pst_bc_c" model="account.report.line">
                <field name="name">C - Commission</field>
                <field name="code">CaBcC</field>
                <!--
                We need to handle very special computation for commissions
                if CaBcB.balance <= 22 then CaBcB.balance
                elif CaBcB.balance <= 333.33 then 22
                elif CaBcB.balance <= 3000 then CaBcB.balance * 6.6%
                else: => 198
                -->
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_pst_bc_c_under_22" model="account.report.expression">
                        <field name="label">under_22</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CaBcB.balance - 22</field>
                        <field name="subformula">if_below(CAD(0))</field>
                    </record>
                    <record id="l10n_ca_tr_pst_bc_c_under_333" model="account.report.expression">
                        <field name="label">under_333</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">(CaBcB.balance - 333.33) * 0.066</field>
                        <field name="subformula">if_above(CAD(0))</field>
                    </record>
                    <record id="l10n_ca_tr_pst_bc_c_over_3000" model="account.report.expression">
                        <field name="label">over_3000</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">-(CaBcB.balance - 3000) * 0.066</field>
                        <field name="subformula">if_below(CAD(0))</field>
                    </record>
                    <record id="l10n_ca_tr_pst_bc_c_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">22 + CaBcC.under_22 + CaBcC.under_333 + CaBcC.over_3000</field>
                        <field name="subformula">if_above(CAD(0))</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_pst_bc_d" model="account.report.line">
                <field name="name">D - Net PST Due on Sales and Leases</field>
                <field name="code">CaBcD</field>
                <field name="aggregation_formula">CaBcB.balance - CaBcC.balance</field>
            </record>
            <record id="l10n_ca_tr_pst_bc_e" model="account.report.line">
                <field name="name">E - Purchase and Lease Price of Taxable Goods, Software and Services from which no PST was paid</field>
                <field name="code">CaBcE</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_pst_bc_g_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaBcE</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_pst_bc_f" model="account.report.line">
                <field name="name">F - PST Due on Purchases and Leases</field>
                <field name="code">CaBcF</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_pst_bc_f_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaBcF</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_pst_bc_g" model="account.report.line">
                <field name="name">G - PST Payable Before Adjustments</field>
                <field name="code">CaBcG</field>
                <field name="aggregation_formula">CaBcD.balance + CaBcF.balance</field>
            </record>
            <record id="l10n_ca_tr_pst_bc_h" model="account.report.line">
                <field name="name">H - PST on Bad Debt Write-Off</field>
                <field name="code">CaBcH</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_pst_bc_h_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaBcH</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_pst_bc_i" model="account.report.line">
                <field name="name">I - PST on Amounts Refunded or Credited to Customers</field>
                <field name="code">CaBcI</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_pst_bc_i_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaBcI</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_pst_bc_j" model="account.report.line">
                <field name="name">J - Total Adjustments</field>
                <field name="code">CaBcJ</field>
                <field name="aggregation_formula">CaBcH.balance + CaBcI.balance</field>
            </record>
            <record id="l10n_ca_tr_pst_bc_k" model="account.report.line">
                <field name="name">K - Total Amount Due</field>
                <field name="code">CaBcK</field>
                <field name="aggregation_formula">CaBcG.balance - CaBcJ.balance</field>
            </record>
        </field>
    </record>

    <record id="l10n_ca_tr_pst_mb" model="account.report">
        <field name="name">Manitoba PST report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ca"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_ca_tr_pst_mb_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_ca_tr_pst_mb_1" model="account.report.line">
                <field name="name">1 - Tax collectable on sales</field>
                <field name="code">CaMb1</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_pst_mb_1_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaMb1</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_pst_mb_2" model="account.report.line">
                <field name="name">2 - Commission</field>
                <field name="code">CaMb2</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <!--
                    We need to handle very special computation for commissions
                    if CaMb1.balance <= 3 then CaMb1.balance
                    elif CaMb1.balance <= 20 then 3
                    elif CaMb1.balance <= 3000 then (max(200, CaMb1.balance) * 15 + min(0, CaMb1.balance-200)) / 100
                    else: => 0
                    -->
                    <record id="l10n_ca_tr_pst_mb_2_under_3" model="account.report.expression">
                        <field name="label">under_3</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CaMb1.balance - 3</field>
                        <field name="subformula">if_below(CAD(0))</field>
                    </record>
                    <record id="l10n_ca_tr_pst_mb_2_under_200" model="account.report.expression">
                        <field name="label">under_200</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">(CaMb1.balance - 20) * 0.15</field>
                        <field name="subformula">if_above(CAD(0))</field>
                    </record>
                    <record id="l10n_ca_tr_pst_mb_2_under_3000" model="account.report.expression">
                        <field name="label">under_3000</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">-(CaMb1.balance - 200) * 0.14</field>
                        <field name="subformula">if_below(CAD(0))</field>
                    </record>
                    <record id="l10n_ca_tr_pst_mb_2_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">3 + CaMb2.under_3 + CaMb2.under_200 + CaMb2.under_3000</field>
                        <field name="subformula">if_between(CAD(0), CAD(58))</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_pst_mb_3" model="account.report.line">
                <field name="name">3 - Tax Owing on Purchases</field>
                <field name="code">CaMb3</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_pst_mb_3_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaMb3</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_pst_mb_35" model="account.report.line">
                <field name="name">Outstanding Balance Including Interest</field>
                <field name="code">CaMb35</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_pst_mb_35_balance" model="account.report.expression">
                   <field name="label">balance</field>
                    <field name="engine">external</field>
                    <field name="formula">sum</field>
                    <field name="subformula">editable;rounding=2</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_pst_mb_4" model="account.report.line">
                <field name="name">4 - Total Amount Due</field>
                <field name="code">CaMb4</field>
                <field name="aggregation_formula">CaMb1.balance - CaMb2.balance + CaMb3.balance + CaMb35.balance</field>
            </record>
        </field>
    </record>

    <record id="l10n_ca_tr_qst" model="account.report">
        <field name="name">Quebec Tax report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ca"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_ca_tr_pst_qst_gsthst" model="account.report.column">
                <field name="name">GST/HST</field>
                <field name="expression_label">gsthst</field>
            </record>
            <record id="l10n_ca_tr_pst_qst_qst" model="account.report.column">
                <field name="name">QST</field>
                <field name="expression_label">qst</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_ca_tr_qst_101" model="account.report.line">
                <field name="name">101 - Total supplies</field>
                <field name="code">CaQc101</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_qst_101_tag" model="account.report.expression">
                        <field name="label">gsthst</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaQc101</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_qst_103_203" model="account.report.line">
                <field name="name">103 - 203 - Tax amounts collectible</field>
                <field name="code">CaQc103203</field>
                <field name="hierarchy_level">5</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_qst_103_tag" model="account.report.expression">
                        <field name="label">gsthst</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaQc103</field>
                    </record>
                    <record id="l10n_ca_tr_qst_203_tag" model="account.report.expression">
                        <field name="label">qst</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaQc203</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_qst_104_204" model="account.report.line">
                <field name="name">104 - 204 - Tax amounts adjustments</field>
                <field name="code">CaQc104204</field>
                <field name="hierarchy_level">5</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_qst_104_tag" model="account.report.expression">
                        <field name="label">gsthst</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaQc104</field>
                    </record>
                    <record id="l10n_ca_tr_qst_204_tag" model="account.report.expression">
                        <field name="label">qst</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaQc204</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_qst_105_205" model="account.report.line">
                <field name="name">105 - 205 - Tax amounts collectible and adjustments</field>
                <field name="code">CaQc105205</field>
                <field name="hierarchy_level">4</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_qst_105_aggr" model="account.report.expression">
                        <field name="label">gsthst</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CaQc103203.gsthst + CaQc104204.gsthst</field>
                    </record>
                    <record id="l10n_ca_tr_qst_205_aggr" model="account.report.expression">
                        <field name="label">qst</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CaQc103203.qst + CaQc104204.qst</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_qst_106_206" model="account.report.line">
                <field name="name">106 - 206 - ITCs/ITRs</field>
                <field name="code">CaQc106206</field>
                <field name="hierarchy_level">5</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_qst_106_tag" model="account.report.expression">
                        <field name="label">gsthst</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaQc106</field>
                    </record>
                    <record id="l10n_ca_tr_qst_206_tag" model="account.report.expression">
                        <field name="label">qst</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaQc206</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_qst_107_207" model="account.report.line">
                <field name="name">107 - 207 - ITCs/ITRs amounts adjustments</field>
                <field name="code">CaQc107207</field>
                <field name="hierarchy_level">5</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_qst_107_tag" model="account.report.expression">
                        <field name="label">gsthst</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaQc107</field>
                    </record>
                    <record id="l10n_ca_tr_qst_207_tag" model="account.report.expression">
                        <field name="label">qst</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaQc207</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_qst_108_208" model="account.report.line">
                <field name="name">108 - 208 - ITCs/ITRs and adjustments</field>
                <field name="code">CaQc108208</field>
                <field name="hierarchy_level">4</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_qst_108_aggr" model="account.report.expression">
                        <field name="label">gsthst</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CaQc106206.gsthst + CaQc107207.gsthst</field>
                    </record>
                    <record id="l10n_ca_tr_qst_208_aggr" model="account.report.expression">
                        <field name="label">qst</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CaQc106206.qst + CaQc107207.qst</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_qst_109_209" model="account.report.line">
                <field name="name">109 - 209 - Net tax amounts</field>
                <field name="code">CaQc109209</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_qst_109_aggr" model="account.report.expression">
                        <field name="label">gsthst</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CaQc105205.gsthst - CaQc108208.gsthst</field>

                    </record>
                    <record id="l10n_ca_tr_qst_209_aggr" model="account.report.expression">
                        <field name="label">qst</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CaQc105205.qst - CaQc108208.qst</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_qst_111_211" model="account.report.line">
                <field name="name">111 - 211 - Tax public service bodies' rebate</field>
                <field name="code">CaQc111211</field>
                <field name="hierarchy_level">3</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_qst_111_tag" model="account.report.expression">
                        <field name="label">gsthst</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaQc111</field>
                    </record>
                    <record id="l10n_ca_tr_qst_211_tag" model="account.report.expression">
                        <field name="label">qst</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaQc211</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_qst_113_213" model="account.report.line">
                <field name="name">113 - 213 - Tax payable or refund claimed</field>
                <field name="code">CaQc113213</field>
                <field name="hierarchy_level">2</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_qst_113_aggr" model="account.report.expression">
                        <field name="label">gsthst</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CaQc109209.gsthst - CaQc111211.gsthst</field>
                    </record>
                    <record id="l10n_ca_tr_qst_213_aggr" model="account.report.expression">
                        <field name="label">qst</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CaQc109209.qst - CaQc111211.qst</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_qst_114_214" model="account.report.line">
                <field name="name">114 - 214 - Tax payable on taxable immovables or taxable carbon emission allowances</field>
                <field name="code">CaQc114214</field>
                <field name="hierarchy_level">2</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_qst_114_tag" model="account.report.expression">
                        <field name="label">gsthst</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaQc114</field>
                    </record>
                    <record id="l10n_ca_tr_qst_214_tag" model="account.report.expression">
                        <field name="label">qst</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaQc214</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_qst_115" model="account.report.line">
                <field name="name">115 - Tax payable on imported taxable supplies</field>
                <field name="code">CaQc115</field>
                <field name="hierarchy_level">2</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_qst_115_tag" model="account.report.expression">
                        <field name="label">gsthst</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaQc115</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_qst_116_216" model="account.report.line">
                <field name="name">116 - 216 - Tax payable or refund claimed</field>
                <field name="code">CaQc116216</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_qst_116_aggr" model="account.report.expression">
                        <field name="label">gsthst</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CaQc113213.gsthst + CaQc114214.gsthst + CaQc115.gsthst</field>
                    </record>
                    <record id="l10n_ca_tr_qst_216_aggr" model="account.report.expression">
                        <field name="label">qst</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">CaQc113213.qst + CaQc114214.qst</field>
                    </record>
                </field>
            </record>
        </field>
    </record>

    <record id="l10n_ca_tr_pst_sk" model="account.report">
        <field name="name">Saskatchewan PST report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ca"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_ca_tr_pst_sk_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_ca_tr_pst_sk_total_sales" model="account.report.line">
                <field name="name">Total Sales Before Taxes</field>
                <field name="code">CaSkTotalSales</field>
                <field name="hierarchy_level">0</field>
                <field name="expression_ids">
                    <record id="l10n_ca_tr_pst_sk_total_sales_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">CaSk8Base</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_pst_sk_step_1" model="account.report.line">
                <field name="name">Step 1</field>
                <field name="code">CaSkStep1</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_ca_tr_pst_sk_1" model="account.report.line">
                        <field name="name">1 - Credits Carried Forward from Previous Period</field>
                        <field name="code">CaSk1</field>
                        <field name="hierarchy_level">3</field>
                        <field name="expression_ids">
                            <record id="l10n_ca_tr_pst_sk_1_balance" model="account.report.expression">
                           <field name="label">balance</field>
                            <field name="engine">external</field>
                            <field name="formula">sum</field>
                            <field name="subformula">editable;rounding=2;if_above(CAD(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ca_tr_pst_sk_2" model="account.report.line">
                        <field name="name">2 - Tax paid on Purchased for Resale</field>
                        <field name="code">CaSk2</field>
                        <field name="hierarchy_level">5</field>
                        <field name="expression_ids">
                            <record id="l10n_ca_tr_pst_sk_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CaSk2</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ca_tr_pst_sk_3" model="account.report.line">
                        <field name="name">3 - Tax Refunded to Customers</field>
                        <field name="code">CaSk3</field>
                        <field name="hierarchy_level">5</field>
                        <field name="expression_ids">
                            <record id="l10n_ca_tr_pst_sk_3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CaSk3</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ca_tr_pst_sk_4" model="account.report.line">
                        <field name="name">4 - Tax Written Off on Bad Debts</field>
                        <field name="code">CaSk4</field>
                        <field name="hierarchy_level">5</field>
                        <field name="expression_ids">
                            <record id="l10n_ca_tr_pst_sk_4_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CaSk4</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ca_tr_pst_sk_5" model="account.report.line">
                        <field name="name">5 - Other</field>
                        <field name="code">CaSk5</field>
                        <field name="hierarchy_level">5</field>
                        <field name="expression_ids">
                            <record id="l10n_ca_tr_pst_sk_5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CaSk5</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ca_tr_pst_sk_6" model="account.report.line">
                        <field name="name">6 - Tax Credits Recorded for this Period</field>
                        <field name="code">CaSk6</field>
                        <field name="hierarchy_level">3</field>
                        <field name="aggregation_formula">CaSk2.balance + CaSk3.balance + CaSk4.balance + CaSk5.balance</field>
                    </record>
                    <record id="l10n_ca_tr_pst_sk_7" model="account.report.line">
                        <field name="name">7 - Tax Credits</field>
                        <field name="code">CaSk7</field>
                        <field name="aggregation_formula">CaSk1.balance + CaSk6.balance</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_pst_sk_step_2" model="account.report.line">
                <field name="name">Step 2</field>
                <field name="code">CaSkStep2</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_ca_tr_pst_sk_8" model="account.report.line">
                        <field name="name">8 - Tax Collected on Sales in this Period</field>
                        <field name="code">CaSk8</field>
                        <field name="hierarchy_level">3</field>
                        <field name="expression_ids">
                            <record id="l10n_ca_tr_pst_sk_8_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CaSk8</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ca_tr_pst_sk_9" model="account.report.line">
                        <field name="name">9 - Portion of Credits Applied</field>
                        <field name="code">CaSk9</field>
                        <field name="hierarchy_level">3</field>
                        <field name="expression_ids">
                            <record id="l10n_ca_tr_pst_sk_9_overflow" model="account.report.expression">
                                <!-- Avoid the CaSk9.balance to go above the value of CaSk8 and CaSk10.balance to go negative -->
                                <field name="label">overflow</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CaSk8.balance - CaSk7.balance</field>
                                <field name="subformula">if_below(CAD(0))</field>
                            </record>
                            <record id="l10n_ca_tr_pst_sk_9_aggr" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CaSk7.balance + CaSk9.overflow</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ca_tr_pst_sk_10" model="account.report.line">
                        <field name="name">10 - Net Tax Collected</field>
                        <field name="code">CaSk10</field>
                        <field name="expression_ids">
                            <record id="l10n_ca_tr_pst_sk_10_aggr" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CaSk8.balance - CaSk9.balance</field>
                                <field name="subformula">if_above(CAD(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ca_tr_pst_sk_11" model="account.report.line">
                        <field name="name">11 - Remaining Tax Credits</field>
                        <field name="code">CaSk11</field>
                        <field name="aggregation_formula">CaSk7.balance - CaSk9.balance</field>
                    </record>
                </field>
            </record>
            <record id="l10n_ca_tr_pst_sk_step_3" model="account.report.line">
                <field name="name">Step 3</field>
                <field name="code">CaSkStep3</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="l10n_ca_tr_pst_sk_12" model="account.report.line">
                        <field name="name">12 - Consumption Tax</field>
                        <field name="code">CaSk12</field>
                        <field name="hierarchy_level">3</field>
                        <field name="expression_ids">
                            <record id="l10n_ca_tr_pst_sk_12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CaSk12</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ca_tr_pst_sk_13" model="account.report.line">
                        <field name="name">13 - Portion of Credits Applied</field>
                        <field name="code">CaSk13</field>
                        <field name="hierarchy_level">3</field>
                        <field name="expression_ids">
                            <record id="l10n_ca_tr_pst_sk_13_overflow" model="account.report.expression">
                                <!-- Avoid the CaSk13.balance to go above the value of CaSk12 and CaSk14.balance to go negative -->
                                <field name="label">overflow</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CaSk12.balance - CaSk11.balance</field>
                                <field name="subformula">if_below(CAD(0))</field>
                            </record>
                            <record id="l10n_ca_tr_pst_sk_13_aggr" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CaSk11.balance + CaSk13.overflow</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ca_tr_pst_sk_14" model="account.report.line">
                        <field name="name">14 - Net Consumption Tax</field>
                        <field name="code">CaSk14</field>
                        <field name="expression_ids">
                            <record id="l10n_ca_tr_pst_sk_14_aggr" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CaSk12.balance - CaSk13.balance</field>
                                <field name="subformula">if_above(CAD(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_ca_tr_pst_sk_15" model="account.report.line">
                        <field name="name">15 - Remaining Tax Credits</field>
                        <field name="code">CaSk15</field>
                        <field name="expression_ids">
                            <record id="l10n_ca_tr_pst_sk_15_aggr" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">CaSk11.balance - CaSk13.balance</field>
                                <field name="subformula">if_above(CAD(0))</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-ca_2023.csv

```csv
"id","code","name","account_type","tag_ids","reconcile","name@fr"
"l10n_ca_111100","111100","Cash in Canadian Currency","asset_cash","","False","Encaisse en monnaie canadienne"
"l10n_ca_111200","111200","Demand Deposits / Notice Deposits in Deposit Accepting Institutions","asset_cash","","False","Dépôts à vue/dépôts à préavis dans les institutions de dépôts"
"l10n_ca_111300","111300","Term Deposits - under 90 days","asset_cash","","False","Dépôts à terme - moins de 90 jours"
"l10n_ca_111400","111400","Other Cash Equivalents","asset_cash","","False","Autres équivalents de trésorerie"
"l10n_ca_111500","111500","Other Current Cash Accounts (including Restricted Cash)","asset_cash","","False","Autres comptes de trésorerie (y compris les liquidités soumises à restrictions)"
"l10n_ca_112100","112100","Accounts Receivable and Accrued Revenue - Gross","asset_receivable","","True","Comptes débiteurs et revenus courus - Brut"
"l10n_ca_112110","112110","Trade Accounts Receivable","asset_receivable","","True","Comptes clients"
"l10n_ca_112113","112113","Customers Account (PoS)","asset_receivable","","True","Comptes clients (PdV)"
"l10n_ca_112120","112120","Rent and Operating Lease Receivable","asset_receivable","","True","Comptes débiteurs - location et location-exploitation"
"l10n_ca_112130","112130","Accrued Interest Receivable and other Accrued Investment Income (non-affiliates)","asset_receivable","","True","Intérêts courus à recevoir et autres revenus de placements à recevoir (sociétés non affiliées)"
"l10n_ca_112131","112131","Interest Receivable and Accrued","asset_receivable","","True","Intérêts courus à recevoir"
"l10n_ca_112132","112132","Investment Income Receivable & Accrued","asset_receivable","","True","Revenus de placements courus à recevoir"
"l10n_ca_112133","112133","Other Accounts Receivables and Accruals (non-affiliates)","asset_receivable","","True","Autres montants et produits à recevoir (sociétés non affiliées)"
"l10n_ca_112140","112140","Finance Receivable (non-affiliates)","asset_receivable","","True","Effets financiers à recevoir (sociétés non affiliées)"
"l10n_ca_112150","112150","Notes Receivable (non-affiliates)","asset_receivable","","True","Billets à recevoir (sociétés non affiliées)"
"l10n_ca_112160","112160","Government assistance receivable","asset_receivable","","True","Aide gouvernementale à recevoir"
"l10n_ca_112210","112210","Allowance for Doubtful Accounts, Trade Accounts Receivable","asset_receivable","","True","Provision pour créances douteuses, comptes débiteurs commerciaux"
"l10n_ca_112220","112220","Allowance for Doubtful Accounts, Rent and Lease Receivable","asset_receivable","","True","Provision pour créances douteuses, créances au titre de baux"
"l10n_ca_112230","112230","Allowance for Doubtful Accounts, Other","asset_receivable","","True","Provision pour créances douteuses, autre"
"l10n_ca_113110","113110","Equity Investment in Canadian Corporations, valued at cost of shares","asset_current","account.account_tag_investing","False","Placement en actions dans des sociétés canadiennes, évalués selon le coût des actions"
"l10n_ca_113120","113120","Equity Investment in Canadian Corporations, valued using Equity Method","asset_current","account.account_tag_investing","False","Placement en actions dans des sociétés canadiennes, évaluées selon la méthode de la comptabilisation à la valeur de consolidation"
"l10n_ca_113210","113210","Equity Investment in foreign Corporations, valued at Cost of Shares","asset_current","account.account_tag_investing","False","Placement en actions dans des sociétés étrangères, évaluées au coût des actions"
"l10n_ca_113220","113220","Equity Investment in foreign Corporations, valued using Equity Method","asset_current","account.account_tag_investing","False","Placement en actions dans des sociétés étrangères, évaluées selon la méthode de la comptabilisation à la valeur de consolidation"
"l10n_ca_113300","113300","Loans, Notes, Accounts, Bonds, Mortgages and Other Investments","asset_current","account.account_tag_investing","False","Prêts, billets, comptes débiteurs, obligations, hypothèques et autres placements"
"l10n_ca_113310","113310","Due from Shareholders, Directors and Officers","asset_current","account.account_tag_investing","False","Sommes exigibles des actionnaires, des administrateurs et des dirigeants"
"l10n_ca_113320","113320","Investments, Loans, Advances, Mortgages, Notes, Bonds and other Claims to Corporate Affiliates and other Related Parties","asset_current","account.account_tag_investing","False","Placements, prêts, avances, hypothèques, billets, obligations et autres réclamations à des sociétés affiliées et autres personnes apparentées"
"l10n_ca_113330","113330","Investment, Loans, Advances and other Claims to in Joint Ventures and Partnerships","asset_current","account.account_tag_investing","False","Placements, prêts, avances et autres créances sur coentreprises et sociétés de personnes"
"l10n_ca_113340","113340","Investment in Government Business Enterprises - Federal, Provincial & Municipal","asset_current","account.account_tag_investing","False","Placements dans des entreprises publiques - fédérales, provinciales et municipales"
"l10n_ca_113350","113350","Investment in Net Assets of Canadian Corp's Unincorporated Branches/Offices Located outside Canada","asset_current","account.account_tag_investing","False","Placements dans l'actif net de succursales non constituées en société d'une société canadienne - bureaux situés à l'étranger"
"l10n_ca_114110","114110","Term Deposits, Treasury Bills and Canada Bills - over 89 days","asset_current","account.account_tag_investing","False","Dépôts à terme, bons du Trésor et bons du Canada - plus de 89 jours"
"l10n_ca_114120","114120","Other Short Term Investment - over 89 days","asset_current","account.account_tag_investing","False","Autres placements à court terme - plus de 89 jours"
"l10n_ca_114130","114130","Bonds & Debentures","asset_current","account.account_tag_investing","False","Obligations et débentures"
"l10n_ca_114210","114210","Corporate Share Capital & Other Equity Instrument","asset_current","account.account_tag_investing","False","Capital-actions et autres instruments de capitaux propres de sociétés"
"l10n_ca_114310","114310","Investments in foreign Non-Affiliates, Debt Instruments","asset_current","account.account_tag_investing","False","Placements dans des sociétés non affiliées étrangères, titres de créance"
"l10n_ca_114320","114320","Investments in Foreign Non-Affiliates, Equity Instruments","asset_current","account.account_tag_investing","False","Placements dans des sociétés non affiliées étrangères, instruments de capitaux propres"
"l10n_ca_114330","114330","Investments in Foreign Non-Affiliates, Other","asset_current","account.account_tag_investing","False","Placements dans des sociétés non affiliées étrangères, autres"
"l10n_ca_115100","115100","Mortgage Loans to Non-Affiliates, Secured by Property in Canada","asset_current","account.account_tag_investing","False","Prêts hypothécaires aux sociétés non affiliées garantis par des propriétés au Canada"
"l10n_ca_115200","115200","Mortgage Loans to Non-Affiliates, Secured by Property outside of Canada","asset_current","account.account_tag_investing","False","Prêts hypothécaires aux sociétés non affiliées garantis par une propriété à l'extérieur du Canada"
"l10n_ca_115300","115300","Allowance for Losses on Mortgage Loans","asset_current","account.account_tag_investing","False","Provision pour pertes sur prêts hypothécaires"
"l10n_ca_116100","116100","Non-Mortgage Loans made by Enterprises in Non-Financial Industries","asset_current","account.account_tag_financing","False","Prêts non hypothécaires consentis par des entreprises non financières"
"l10n_ca_116110","116110","Loans Receivable","asset_current","account.account_tag_financing","False","Comptes débiteurs au titre de prêts"
"l10n_ca_116120","116120","Finance Lease Contracts Receivable","asset_current","account.account_tag_financing","False","Comptes débiteurs au titre de contrats de location-financement"
"l10n_ca_116200","116200","Allowance for Losses on Non-Mortgage Loans","asset_current","account.account_tag_financing","False","Provision pour pertes sur prêts non hypothécaires"
"l10n_ca_116210","116210","Allowance for Losses, Loans","asset_current","account.account_tag_financing","False","Provision pour pertes sur prêts"
"l10n_ca_116220","116220","Allowance for Doubtful Accounts, Capital/Financial Lease Contracts","asset_current","account.account_tag_financing","False","Provisions pour créances douteuses, contrats de location-acquisition ou de crédit-bail"
"l10n_ca_117000","117000","Derivative Assets","asset_current","account.account_tag_investing","False","Actifs dérivés"
"l10n_ca_118100","118100","GST receivable","asset_current","","False","TPS recevable"
"l10n_ca_118200","118200","PST/QST receivable","asset_current","","False","TVP/TVQ à recevoir"
"l10n_ca_118300","118300","HST receivable - 13%","asset_current","","False","TVH à recevoir - 13%"
"l10n_ca_118500","118500","HST receivable - 15%","asset_current","","False","TVH à recevoir - 15%"
"l10n_ca_121110","121110","Manufacturing Inventories","asset_current","","False","Stocks manufacturiers"
"l10n_ca_121111","121111","Manufacturing Inventories, Raw Material","asset_current","","False","Stocks manufacturiers, matière première"
"l10n_ca_121112","121112","Manufacturing Inventories, Work in Progress","asset_current","","False","Stocks manufacturiers, travaux en cours"
"l10n_ca_121120","121120","Goods for Sale Inventory","asset_current","","False","Stock de biens destinés à la vente"
"l10n_ca_121121","121121","Goods for Sale Inventory, Manufacturing Finished Goods","asset_current","","False","Stock de biens destinés à la vente, fabrication de produits finis"
"l10n_ca_121122","121122","Goods for Sale Inventory, Goods Purchased for Resale","asset_current","","False","Stock de biens destinés à la vente, biens achetés pour la revente"
"l10n_ca_121123","121123","Goods for Sale Inventory, Retail Trade","asset_current","","False","Stock de biens destinés à la vente, commerce de détail"
"l10n_ca_121124","121124","Goods for Sale Inventory, Wholesale Trade","asset_current","","False","Stock de biens destinés à la vente, commerce de gros"
"l10n_ca_121125","121125","Goods for Sale Inventory, Gold","asset_current","","False","Stock de biens destinés à la vente, or"
"l10n_ca_121126","121126","Goods for Sale Inventory, Other Non-Farm Inventories","asset_current","","False","Stock de biens destinés à la vente, autres stocks non agricoles"
"l10n_ca_121127","121127","Goods for Sale Inventory, Farm Grain","asset_current","","False","Stock de biens destinés à la vente, céréales"
"l10n_ca_121128","121128","Goods for Sale Inventory, Farm Non-Grain Inventories","asset_current","","False","Stock de biens destinés à la vente, stocks agricoles autres que les céréales"
"l10n_ca_121130","121130","Stock Received But Not Billed","asset_current","","True","Marchandises reçues non-facturées"
"l10n_ca_121140","121140","Stock Delivered But Not Billed","asset_current","","True","Marchandises expédiées non-facturées"
"l10n_ca_121200","121200","Part and Supplies Inventories","asset_current","account.account_tag_investing","False","Stocks de pièces et fournitures"
"l10n_ca_121310","121310","Real Estate Held or being Developed for Sale, Residential","asset_current","account.account_tag_investing","False","Biens immobiliers détenus ou en cours d'aménagement en vue de la vente, résidentiels"
"l10n_ca_121320","121320","Real Estate Held or being Developed for Sale, Non-Residential","asset_current","account.account_tag_investing","False","Biens immobiliers détenus ou en cours d'aménagement en vue de la vente, non résidentiels"
"l10n_ca_122000","122000","Allowance for Obsolescence and decline in Value","asset_current","","False","Provision pour obsolescence et dévaluation"
"l10n_ca_123000","123000","Breeding Livestock","asset_current","","False","Animaux de reproduction"
"l10n_ca_124100","124100","Fixed Assets - Net","asset_fixed","","False","Immobilisations corporelles - Nets"
"l10n_ca_124101","124101","Land","asset_fixed","","False","Terrain"
"l10n_ca_124102","124102","Buildings and Other Structures Under Construction","asset_fixed","","False","Bâtiments et autres ouvrages en construction"
"l10n_ca_124103","124103","Land Improvements","asset_fixed","","False","Aménagements de terrains"
"l10n_ca_124104","124104","Buildings","asset_fixed","","False","Bâtiments"
"l10n_ca_124105","124105","Leasehold Improvements","asset_fixed","","False","Améliorations locatives"
"l10n_ca_124106","124106","All other Structures","asset_fixed","","False","Autres structures"
"l10n_ca_124107","124107","Machinery and Equipment","asset_fixed","","False","Machines et matériel"
"l10n_ca_124108","124108","Capital Leases","asset_fixed","","False","Contrats de location-acquisition"
"l10n_ca_124109","124109","Land Improvements - Accumulated Amortization","asset_fixed","","False","Aménagements de terrains - Amortissements cumulés"
"l10n_ca_124110","124110","Buildings - Accumulated Amortization","asset_fixed","","False","Bâtiments - Amortissements cumulés"
"l10n_ca_124111","124111","Leasehold Improvements - Accumulated Amortization","asset_fixed","","False","Améliorations locatives - Amortissements cumulés"
"l10n_ca_124112","124112","All Other Structures - Accumulated Amortization","asset_fixed","","False","Autres structures - Amortissements cumulés"
"l10n_ca_124113","124113","Machinery and Equipment - Accumulated Amortization","asset_fixed","","False","Machines et matériel - Amortissements cumulés"
"l10n_ca_124114","124114","Capital Leases - Accumulated Amortization","asset_fixed","","False","Contrats de location-acquisition - Amortissements cumulés"
"l10n_ca_124200","124200","Depletable Assets - Net","asset_fixed","","False","Actifs non renouvelables - nets"
"l10n_ca_124210","124210","Depletable Assets - Gross","asset_fixed","","False","Actifs non renouvelables - bruts"
"l10n_ca_124220","124220","Depletable Assets - Accumulated Amortization","asset_fixed","","False","Actifs non renouvelables - Amortissements cumulés"
"l10n_ca_124300","124300","Repossessed Assets Held for Sale","asset_fixed","","False","Reprise de possession d'immobilisations destinées à la vente"
"l10n_ca_131100","131100","Deferred Charges - Net","asset_prepayments","","False","Frais reportés - nets"
"l10n_ca_131110","131110","Deferred Charges (expenses capitalized) - Gross","asset_prepayments","","False","Frais reportés (frais immobilisés) - bruts"
"l10n_ca_131120","131120","Deferred Charges (expenses capitalized) - Accumulated Amortization","asset_prepayments","","False","Frais reportés (frais immobilisés) - Amortissements cumulés"
"l10n_ca_131200","131200","Prepaid Expenses","asset_prepayments","","False","Frais payés d'avance"
"l10n_ca_131210","131210","Drilling Advances","asset_prepayments","","False","Avances de forage"
"l10n_ca_131220","131220","Other Prepaid Expenses","asset_prepayments","","False","Autres frais payées d'avance"
"l10n_ca_132100","132100","Goodwill - Net","asset_non_current","","False","Achalandage - net"
"l10n_ca_132110","132110","Goodwill - Gross","asset_non_current","","False","Achalandage - brut"
"l10n_ca_132120","132120","Goodwill - Accumulated Amortization","asset_non_current","","False","Achalandage - Amortissements cumulés"
"l10n_ca_132200","132200","Software","asset_non_current","","False","Logiciels"
"l10n_ca_132300","132300","Database","asset_non_current","","False","Bases de données"
"l10n_ca_132400","132400","Excess of Purchase Price of Consolidated Subsidiary Shares","asset_non_current","","False","Excédent du prix d'achat d'actions d'une filiale consolidée"
"l10n_ca_132500","132500","Patents, Trademarks and Copyrights - Net","asset_non_current","","False","Brevets, marques de commerce et droits d'auteurs - nets"
"l10n_ca_132510","132510","Patents, Trademarks and Copyrights - Gross","asset_non_current","","False","Brevets, marques de commerce et droits d'auteurs - bruts"
"l10n_ca_132520","132520","Patents, Trademarks and Copyrights - Accumulated Amortization","asset_non_current","","False","Brevets, marques de commerce et droits d'auteurs - Amortissements cumulés"
"l10n_ca_132600","132600","Other Intangibles","asset_non_current","","False","Autres actifs incorporels"
"l10n_ca_132610","132610","Quota - Net","asset_non_current","","False","Contingents - nets"
"l10n_ca_132611","132611","Quota - Gross","asset_non_current","","False","Contingents - bruts"
"l10n_ca_132612","132612","Quota - Accumulated Amortization","asset_non_current","","False","Contingents - Amortissements cumulés"
"l10n_ca_132620","132620","License - Net","asset_non_current","","False","Permis - nets"
"l10n_ca_132621","132621","License - Gross","asset_non_current","","False","Permis - bruts"
"l10n_ca_132622","132622","License - Accumulated Amortization","asset_non_current","","False","Permis - Amortissements cumulés"
"l10n_ca_132630","132630","Incorporation Cost - Net","asset_non_current","","False","Frais de constitution - nets"
"l10n_ca_132631","132631","Incorporation Cost - Gross","asset_non_current","","False","Frais de constitution - bruts"
"l10n_ca_132632","132632","Incorporation Cost - Accumulated Amortization","asset_non_current","","False","Frais de constitution - Amortissements cumulés"
"l10n_ca_132640","132640","Customer List - Net","asset_non_current","","False","Liste de clients - nets"
"l10n_ca_132641","132641","Customer List - Gross","asset_non_current","","False","Liste de clients - bruts"
"l10n_ca_132642","132642","Customer List - Accumulated Amortization","asset_non_current","","False","Liste de clients - Amortissements cumulés"
"l10n_ca_132650","132650","Right - Net","asset_non_current","","False","Droits - nets"
"l10n_ca_132651","132651","Right - Gross","asset_non_current","","False","Droits - bruts"
"l10n_ca_132652","132652","Right - Accumulated Amortization","asset_non_current","","False","Droits - Amortissements cumulés"
"l10n_ca_132660","132660","Timber Rights - Net","asset_non_current","","False","Droits de coupe - nets"
"l10n_ca_132661","132661","Timber Rights - Gross","asset_non_current","","False","Droits de coupe - bruts"
"l10n_ca_132662","132662","Timber Rights - Accumulated Amortization","asset_non_current","","False","Droits de coupe - Amortissements cumulés"
"l10n_ca_132670","132670","Mining Right - Net","asset_non_current","","False","Droits miniers - nets"
"l10n_ca_132671","132671","Mining Right - Gross","asset_non_current","","False","Droits miniers - bruts"
"l10n_ca_132672","132672","Mining Right - Accumulated Amortization","asset_non_current","","False","Droits miniers - Amortissements cumulés"
"l10n_ca_132680","132680","Oil and Gas Right - Net","asset_non_current","","False","Droits pétroliers et gaziers - nets"
"l10n_ca_132681","132681","Oil and Gas Right - Gross","asset_non_current","","False","Droits pétroliers et gaziers - bruts"
"l10n_ca_132682","132682","Oil and Gas Right - Accumulated Amortization","asset_non_current","","False","Droits pétroliers et gaziers - Amortissements cumulés"
"l10n_ca_132690","132690","All Other Intangibles Assets - Net","asset_non_current","","False","Tous autres actifs incorporels - nets"
"l10n_ca_132691","132691","All Other Intangibles Assets - Gross","asset_non_current","","False","Tous autres actifs incorporels - bruts"
"l10n_ca_132692","132692","All Other Intangibles Assets - Accumulated Amortization","asset_non_current","","False","Tous autres actifs incorporels - Amortissements cumulés"
"l10n_ca_141100","141100","Pensions Costs (Debits)","asset_non_current","","False","Frais liés aux régimes de retraite (débits)"
"l10n_ca_141200","141200","Actuarial Surplus on Employer-Sponsored Trustee Pension Plans","asset_non_current","","False","Excédent actuariel des régimes de retraite fiduciaires d'employeurs"
"l10n_ca_142110","142110","Deposits with Suppliers","asset_non_current","","False","Dépôts chez les fournisseurs"
"l10n_ca_142120","142120","Security Tender Deposits","asset_non_current","","False","Cautionnement, dépôts sur soumissions"
"l10n_ca_142130","142130","Deposits with Others","asset_non_current","","False","Dépôts chez d'autres entités"
"l10n_ca_142210","142210","Reserve Fund","asset_non_current","","False","Fonds de réserve"
"l10n_ca_142220","142220","Cash Surrender Value of Life Insurance","asset_non_current","","False","Valeur de rachat de l'assurance vie"
"l10n_ca_142230","142230","Asset Held in Trust","asset_non_current","","False","Éléments d'actif détenus en fiducie"
"l10n_ca_142240","142240","Other assets","asset_non_current","","False","Autre actifs"
"l10n_ca_142241","142241","Gold, silver and other precious metal certificates","asset_non_current","","False","Certificats - or, argent et autres métaux précieux"
"l10n_ca_142242","142242","Valuable and Collectibles (eg. Art)","asset_non_current","","False","Objets de valeur et de collection (p. ex., art)"
"l10n_ca_142243","142243","Net Income Stabilization Account","asset_non_current","","False","Compte de stabilisation du revenu net"
"l10n_ca_142244","142244","Canadian Agricultural Income Stabilization Program","asset_non_current","","False","Programme canadien de stabilisation du revenu agricole"
"l10n_ca_142245","142245","Agricultural Income Stabilization Program (Quebec Province only)","asset_non_current","","False","Compte de stabilisation du revenu agricole (province de Québec seulement)"
"l10n_ca_142246","142246","Other Miscellaneous","asset_non_current","","False","Autres objets divers"
"l10n_ca_211110","211110","Residential Mortgage Loans, Secured by Property in Canada","liability_non_current","account.account_tag_financing","False","Prêts hypothécaires résidentiels, garantis par une propriété au Canada"
"l10n_ca_211120","211120","Residential Mortgage Loans, Secured by Property outside of Canada","liability_non_current","account.account_tag_financing","False","Prêts hypothécaires résidentiels, garantis par une propriété à l'extérieur du Canada"
"l10n_ca_211210","211210","Non-Residential Mortgage Loans, Secured by Property in Canada","liability_non_current","account.account_tag_financing","False","Prêts hypothécaires non résidentiels, garantis par une propriété au Canada"
"l10n_ca_211220","211220","Non-Residential Mortgage Loans, Secured by Property outside of Canada","liability_non_current","account.account_tag_financing","False","Prêts hypothécaires non résidentiels, garantis par une propriété à l'extérieur du Canada"
"l10n_ca_212100","212100","Non-Mortgage Loans, Bank Loans - Overdrafts and Loans from Chartered Banks and Branches of Foreign Banks Operating in Canada (excluding Lien Notes Payable)","liability_current","account.account_tag_financing","False","Prêts non hypothécaires et prêts de banque - Découverts et prêts auprès de banques à charte et de succursales de banques étrangères établies au Canada (à l'exclusion des billets portant privilège)"
"l10n_ca_212110","212110","Overdrafts with Chartered Bank Branches in Canada","liability_current","account.account_tag_financing","False","Découverts dans des succursales de banques à charte au Canada"
"l10n_ca_212120","212120","Credit Card Balances","liability_current","account.account_tag_financing","False","Solde de cartes de crédit"
"l10n_ca_212130","212130","Subordinated Loans and Notes Payable","liability_current","account.account_tag_financing","False","Prêts subordonnés et billets à payer"
"l10n_ca_212140","212140","Line of Credit","liability_current","account.account_tag_financing","False","Marge de crédit"
"l10n_ca_212150","212150","Obligations under Financial/Capital Leases","liability_current","account.account_tag_financing","False","Obligations en vertu d'un contrat de location-acquisition ou d'un crédit-bail"
"l10n_ca_212160","212160","Loans with Chartered Bank Branches in Canada","liability_current","account.account_tag_financing","False","Prêts dans des succursales de banques à charte au Canada"
"l10n_ca_212170","212170","Loans from Chartered Bank Branches outside of Canada","liability_current","account.account_tag_financing","False","Prêts auprès de succursales de banques à charte à l'extérieur du Canada"
"l10n_ca_212200","212200","Non-Mortgage Loans, Loans from Other Domestic Lending Institutions","liability_current","account.account_tag_financing","False","Prêts non hypothécaires, prêts auprès d'autres sociétés de financement canadiennes"
"l10n_ca_212210","212210","Credit Union/Caisse Populaire Loans","liability_current","account.account_tag_financing","False","Prêts auprès de caisses d'épargne et de crédit ou de caisses populaires"
"l10n_ca_212220","212220","Farm Credit Corporation Loan Payable","liability_current","account.account_tag_financing","False","Prêts auprès d'une société de crédit agricole à payer"
"l10n_ca_212230","212230","Central League/Federation Loans","liability_current","account.account_tag_financing","False","Prêts de la fédération des ligues centrales"
"l10n_ca_212240","212240","Loans from other Domestic Lending Financial Institutions","liability_current","account.account_tag_financing","False","Prêts auprès d'autres sociétés de financement canadiennes"
"l10n_ca_212300","212300","Non-Mortgage Loans, Loans from Others","liability_current","account.account_tag_financing","False","Prêts non hypothécaires, prêts auprès d'autres entités"
"l10n_ca_212310","212310","Loans from Provincial Governments","liability_current","account.account_tag_financing","False","Prêts auprès des gouvernements provinciaux"
"l10n_ca_212320","212320","Loans from Supply Companies","liability_current","account.account_tag_financing","False","Prêts auprès des sociétés approvisionneuses"
"l10n_ca_212330","212330","Private Loans","liability_current","account.account_tag_financing","False","Prêts personnels"
"l10n_ca_212340","212340","Other Domestic Loans","liability_current","account.account_tag_financing","False","Autres prêts canadiens"
"l10n_ca_212350","212350","Loans from Non-Residents","liability_current","account.account_tag_financing","False","Prêts de non-résidents"
"l10n_ca_212400","212400","Non-Mortgage Loans, Lien Note Payable","liability_current","account.account_tag_financing","False","Prêts non hypothécaires, billets portant privilège à payer"
"l10n_ca_231410","231410","Lien Note Payable to Chartered Bank Branches in Canada","liability_current","account.account_tag_financing","False","Billets portant privilège à payer à des succursales de banque à charte au Canada"
"l10n_ca_212420","212420","Lien Note Payable to Bank Branches Outside Canada","liability_current","account.account_tag_financing","False","Billets portant privilège à payer à des succursales de banque à l'étranger"
"l10n_ca_212430","212430","Lien Note Payable to Other Financial Institutions","liability_current","account.account_tag_financing","False","Billets portant privilège à payer à d'autres institutions financières"
"l10n_ca_212510","212510","Obligations under Financial/Capital Leases (excluding from Chartered Banks), deferred","liability_current","account.account_tag_financing","False","Obligations (autres qu'envers les banques à charte) - contrats de location-acquisition ou crédit-bail, différé"
"l10n_ca_212520","212520","Obligations under Financial/Capital Leases (excluding from Chartered Banks), current","liability_current","account.account_tag_financing","False","Obligations (autres qu'envers les banques à charte) - contrats de location-acquisition ou crédit-bail, courant"
"l10n_ca_213110","213110","Commercial Paper","liability_current","account.account_tag_financing","False","Effets de commerce"
"l10n_ca_213120","213120","Finance Company Paper","liability_current","account.account_tag_financing","False","Effets de sociétés de financement"
"l10n_ca_213130","213130","Bankers' Acceptances","liability_current","account.account_tag_financing","False","Acceptations bancaires"
"l10n_ca_213210","213210","Promissory Notes","liability_current","account.account_tag_financing","False","Billets"
"l10n_ca_213220","213220","Corporate Bonds and Debentures","liability_current","account.account_tag_financing","False","Obligations et débentures de société"
"l10n_ca_213230","213230","Asset-Backed Securities","liability_current","account.account_tag_financing","False","Titres adossés à des crédits mobiliers"
"l10n_ca_214100","214100","Derivative Liabilities With Resident Counterparties","liability_current","account.account_tag_financing","False","Passifs dérivés avec contreparties résidentes"
"l10n_ca_214200","214200","Derivative Liabilities With Non-Resident Counterparties","liability_current","account.account_tag_financing","False","Passifs dérivés avec contreparties non résidentes"
"l10n_ca_215000","215000","Current portion of long-term debt","liability_current","account.account_tag_financing","False","Partie à court terme de la dette à long terme"
"l10n_ca_221110","221110","Accounts Payable - Retail and Wholesale Trade Accounts","liability_payable","account.account_tag_financing","True","Comptes créditeurs - Comptes de commerce de gros et de détail"
"l10n_ca_221120","221120","Accounts Payable - Other Trade Accounts","liability_payable","account.account_tag_financing","True","Comptes créditeurs - Autres comptes commerciaux"
"l10n_ca_221130","221130","Accounts Payable - Holdback Payable","liability_payable","account.account_tag_financing","True","Comptes créditeurs - Retenues de garantie à payer"
"l10n_ca_221200","221200","Other Accounts Payable and Accruals (Non-Affiliates)","liability_payable","account.account_tag_financing","True","Autres comptes créditeurs et charges à payer (sociétés non affiliées)"
"l10n_ca_221201","221201","Interest Payable and Accrued","liability_payable","account.account_tag_financing","True","Intérêts courus à payer"
"l10n_ca_221202","221202","Amounts Payable to Members of Non-Profit Organization","liability_payable","account.account_tag_financing","True","Montants à payer aux membres d'organismes sans but lucratif"
"l10n_ca_221203","221203","Wage payable and Accrued","liability_payable","account.account_tag_financing","True","Salaires courus à payer"
"l10n_ca_221204","221204","Management Fee Payable and Accrued","liability_payable","account.account_tag_financing","True","Frais de gestion courus à payer"
"l10n_ca_221205","221205","Bonus Payable and Accrued","liability_payable","account.account_tag_financing","True","Gratifications courues à payer"
"l10n_ca_221206","221206","Employee Deduction Payable","liability_payable","account.account_tag_financing","True","Retenues salariales à payer"
"l10n_ca_221207","221207","Items in Transit and Outstanding Cheque","liability_payable","account.account_tag_financing","True","Effets en cours de compensation et chèques en circulation"
"l10n_ca_221208","221208","Payable Dividend","liability_payable","account.account_tag_financing","True","Dividendes à payer"
"l10n_ca_221209","221209","Fishing Crew Shares","liability_payable","account.account_tag_financing","True","Actions dans une équipe de pêche"
"l10n_ca_221210","221210","Owing to Governments","liability_payable","account.account_tag_financing","True","Sommes dues aux administrations publiques"
"l10n_ca_221211","221211","Other Accounts Payable and Accrued Liabilities","liability_payable","account.account_tag_financing","True","Autres comptes créditeurs et charges à payer"
"l10n_ca_222100","222100","Amounts Owing to Shareholders, Directors and Officers","liability_payable","account.account_tag_financing","True","Sommes dues aux actionnaires, aux administrateurs et aux dirigeants"
"l10n_ca_222200","222200","Amounts Owing to Parents, Subsidiaries, Related Corporations, Joint Ventures and Partnerships","liability_payable","account.account_tag_financing","True","Sommes dues à la société mère, aux filiales, aux sociétés liées, aux coentreprises et aux sociétés de personnes"
"l10n_ca_222300","222300","Claims of Governments and Associated Government Business Enterprises (Government Business Enterprises only)","liability_payable","account.account_tag_financing","True","Créances des administrations publiques et des entreprises associées aux administrations publiques (entreprises publiques seulement)"
"l10n_ca_231000","231000","GST to pay","liability_current","","False","TPS payable"
"l10n_ca_232000","232000","PST/QST to pay","liability_current","","False","TVP/TVQ à payer"
"l10n_ca_233000","233000","HST to pay - 13%","liability_current","","False","TVH à payer - 13%"
"l10n_ca_235000","235000","HST to pay - 15%","liability_current","","False","TVH à payer - 15%"
"l10n_ca_241000","241000","Future Income Tax, Current","liability_current","","False","Passifs d'impôts futurs, à court terme"
"l10n_ca_242000","242000","Future Income Tax, Long-Term","liability_non_current","","False","Passifs d'impôts futurs, à long terme"
"l10n_ca_261000","261000","Pension Plans, Trustee Staff Pension Plans - Actuarial deficit","liability_non_current","","False","Régimes de retraite, régimes de retraite en fiducie - Déficit actuariel"
"l10n_ca_262000","262000","Pension Plans, Non-Trustee Staff Pension Plans - Actuarial deficit","liability_non_current","","False","Régimes de retraite, régimes de retraite non gérés en fiducie - Déficit actuariel"
"l10n_ca_263000","263000","Pension Plans, Staff Deferred profit-sharing Pension Plans","liability_non_current","","False","Régimes de retraite, régimes de participation différée aux bénéfices"
"l10n_ca_264000","264000","Other Pension plan liabilities and charges","liability_non_current","","False","Autres passifs relatifs aux régimes de retraite"
"l10n_ca_265000","265000","Amounts Owed Due to Other Retirement Plans Obligations","liability_non_current","","False","Sommes exigibles résultant d'autres obligations de régimes de retraite"
"l10n_ca_271000","271000","Reserves for Guarantees, Warranties & Indemnities","liability_non_current","","False","Réserves pour garanties et indemnités"
"l10n_ca_272000","272000","Restructuring Allowance","liability_non_current","","False","Provision pour restructuration"
"l10n_ca_273000","273000","Litigation Allowance","liability_non_current","","False","Provision pour frais de contentieux"
"l10n_ca_274000","274000","Commitments & Contingencies","liability_non_current","","False","Engagements et éventualités"
"l10n_ca_275000","275000","Provision for major overhauls","liability_non_current","","False","Provision pour gros travaux de remise en état"
"l10n_ca_276000","276000","Provisions for Site Restoration","liability_non_current","","False","Provision pour restauration d'un site"
"l10n_ca_277000","277000","Other provisions and reserves","liability_non_current","","False","Autres provisions et réserves"
"l10n_ca_291100","291100","Deferred Gains on Translation of Foreign Currency","liability_non_current","","False","Gains reportés sur conversion de devises"
"l10n_ca_291200","291200","Deferred Losses on Translation of Foreign Currency","liability_non_current","","False","Pertes reportés sur conversion de devises"
"l10n_ca_292000","292000","Contributions to Environmental Trust","liability_non_current","","False","Contributions à une fiducie pour l'environnement"
"l10n_ca_293000","293000","Restated Preference Shares","liability_non_current","","False","Actions privilégiées ajustées"
"l10n_ca_294000","294000","Amounts Held in Trust","liability_non_current","","False","Montants détenus en fiducie"
"l10n_ca_295000","295000","Stock Appreciation Rights - settled in cash","liability_non_current","","False","Droits à la plus-value d'actions - réglés en espèces"
"l10n_ca_296000","296000","Liabilities of disposal groups classified as held for sale","liability_non_current","","False","Passifs des groupes à céder classés comme détenus en vue de la vente"
"l10n_ca_297000","297000","All Other Non-Current Liabilities","liability_non_current","","False","Tous les autres éléments de passif non-courants"
"l10n_ca_298000","298000","All Other Current Liabilities","liability_current","","False","Tous les autres éléments de passif courants"
"l10n_ca_311100","311100","Preferred Shares - New issues","equity","account.account_tag_financing","False","Actions privilégiées - nouvelles émissions"
"l10n_ca_311200","311200","Preferred Shares - New issues (Dividend Reinvestment Plans)","equity","account.account_tag_financing","False","Actions privilégiées - nouvelles émissions (plans de réinvestissement des dividendes)"
"l10n_ca_311300","311300","Preferred Shares - Share Buybacks & Redemptions","equity","account.account_tag_financing","False","Actions privilégiées - rachats et remboursements"
"l10n_ca_312100","312100","Common Shares - New issues","equity","account.account_tag_financing","False","Actions ordinaires - nouvelles émissions"
"l10n_ca_312200","312200","Common Shares - New issues (Dividend Reinvestment plans)","equity","account.account_tag_financing","False","Actions ordinaires - nouvelles émissions (plans de réinvestissement des dividendes)"
"l10n_ca_312300","312300","Common Shares - Share Buybacks & Redemptions","equity","account.account_tag_financing","False","Actions ordinaires - rachats et remboursements"
"l10n_ca_313000","313000","Owners' Equity (Owners' Investment - No Share Capital Entities)","equity","account.account_tag_financing","False","Avoir propre (mise de fonds du propriétaire - entités sans capital-actions)"
"l10n_ca_314000","314000","Other Shares","equity","account.account_tag_financing","False","Autres actions"
"l10n_ca_321000","321000","Retained Earnings/Deficits, Appropriated","equity","account.account_tag_financing","False","Bénéfices non répartis / déficits, affectés"
"l10n_ca_322000","322000","Retained Earnings/Deficits, Unappropriated","equity_unaffected","account.account_tag_financing","False","Bénéfices non répartis / déficits, non affectés"
"l10n_ca_330000","330000","Contributed Surplus","equity","account.account_tag_financing","False","Surplus d'apport"
"l10n_ca_341000","341000","Currency Translation Adjustment for Self Sustaining Foreign Operations","equity","account.account_tag_financing","False","Écart de conversion - entités autonomes à l'étranger"
"l10n_ca_342000","342000","Warrants & Rights Outstanding","equity","account.account_tag_financing","False","Bons de souscription et droits en circulation"
"l10n_ca_343000","343000","Stock Appreciation Rights settled by Issuance of Shares","equity","account.account_tag_financing","False","Droits à la plus-value d'actions - réglés par émission d'actions"
"l10n_ca_344000","344000","Asset Appraisal Increase/Decrease","equity","account.account_tag_financing","False","Plus-value ou moins-value de réévaluation de l'actif"
"l10n_ca_345000","345000","Miscellaneous - all others","equity","account.account_tag_financing","False","Divers"
"l10n_ca_350000","350000","Non-controlling interests","equity","account.account_tag_financing","False","Intérêts minoritaires"
"l10n_ca_360000","360000","Reserves","equity","account.account_tag_financing","False","Réserves"
"l10n_ca_411000","411000","Sales of Goods and Services","income","account.account_tag_operating","False","Ventes de biens et de services"
"l10n_ca_411100","411100","Sales of Goods","income","account.account_tag_operating","False","Ventes de biens"
"l10n_ca_411110","411110","Sales of Goods Purchased for Resale","income","account.account_tag_operating","False","Ventes de biens achetés pour la revente"
"l10n_ca_411120","411120","Sales of Goods Produced","income","account.account_tag_operating","False","Ventes de biens produits"
"l10n_ca_411200","411200","Sales of Services","income","account.account_tag_operating","False","Ventes de services"
"l10n_ca_411210","411210","Commissions and Fees","income","account.account_tag_operating","False","Commissions et honoraires"
"l10n_ca_411220","411220","Rental and Leasing Revenue","income","account.account_tag_operating","False","Revenus de location et de crédit-bail"
"l10n_ca_411221","411221","Subrental and Subleasing Revenue","income","account.account_tag_operating","False","Revenus de sous-location et de crédit-bail partiel"
"l10n_ca_411230","411230","Sales of Services Excluding Commission & Rental","income","account.account_tag_operating","False","Ventes de services, commissions et locations exclues"
"l10n_ca_413100","413100","Subsidies and Grants","income","account.account_tag_operating","False","Subventions et octrois"
"l10n_ca_413200","413200","Royalties and Franchise Fees Revenue","income","account.account_tag_operating","False","Redevances de franchisage et autres redevances"
"l10n_ca_413210","413210","Royalties","income","account.account_tag_operating","False","Redevances"
"l10n_ca_413220","413220","Franchise Fees","income","account.account_tag_operating","False","Redevances de franchisage"
"l10n_ca_413300","413300","All Other Operating Revenue","income","account.account_tag_operating","False","Autres revenus d'exploitation"
"l10n_ca_413310","413310","Intracompany revenue eliminated on consolidation, including transfers between other business units","income","account.account_tag_operating","False","Revenus intersociétés éliminés lors de la consolidation, y compris les transferts entre d'autres unités de l'entreprise"
"l10n_ca_413320","413320","All Other","income","account.account_tag_operating","False","Autres frais"
"l10n_ca_421100","421100","Dividends from Canadian Corporations","income","account.account_tag_operating","False","Dividendes de sociétés canadiennes"
"l10n_ca_421200","421200","Dividends from Foreign Corporations","income","account.account_tag_operating","False","Dividendes de sociétés étrangères"
"l10n_ca_422100","422100","Interest from Canadian Sources","income","account.account_tag_operating","False","Intérêts de source canadienne"
"l10n_ca_422200","422200","Interest from Foreign Sources","income","account.account_tag_operating","False","Intérêts de sources étrangères"
"l10n_ca_423000","423000","Other Non-operating Revenue","income","account.account_tag_operating","False","Autres revenus non liés à l'exploitation"
"l10n_ca_423100","423100","Cash Discount Gain","income","account.account_tag_operating","False","Gain d'escompte"
"l10n_ca_423200","423200","Discounts Taken","income","account.account_tag_operating","False","Réductions reçues"
"l10n_ca_511100","511100","Opening Inventories","expense_direct_cost","account.account_tag_operating","False","Stocks d'ouverture"
"l10n_ca_511110","511110","Raw Materials and Components, Including Non-returnable Containers and Other Shipping and Packaging Materials","expense_direct_cost","account.account_tag_operating","False","Matières premières et composantes, y compris les emballages à usage unique et autres matériaux d'emballage et d'expédition"
"l10n_ca_511120","511120","Goods/Work in Process","expense_direct_cost","account.account_tag_operating","False","Biens en cours de fabrication ou travaux en cours"
"l10n_ca_511130","511130","Finished Goods","expense_direct_cost","account.account_tag_operating","False","Produits finis"
"l10n_ca_511131","511131","Goods Manufactured","expense_direct_cost","account.account_tag_operating","False","Produits fabriqués"
"l10n_ca_511132","511132","Goods Purchased for Resale (as is)","expense_direct_cost","account.account_tag_operating","False","Biens achetés pour la revente (tels quels)"
"l10n_ca_511140","511140","Other Inventories","expense_direct_cost","account.account_tag_operating","False","Autres stocks"
"l10n_ca_511200","511200","Purchases and Costs Chargeable to Cost of Sales","expense_direct_cost","account.account_tag_operating","False","Achats et coûts imputables au coût des marchandises vendues"
"l10n_ca_511210","511210","Purchases","expense_direct_cost","account.account_tag_operating","False","Achats"
"l10n_ca_511211","511211","Purchases of Raw Materials and Components, Including Freight","expense_direct_cost","account.account_tag_operating","False","Achats de matières premières et de composantes, transport compris"
"l10n_ca_511212","511212","Purchases of Non-returnable Containers and Other Shipping and Packaging Materials","expense_direct_cost","account.account_tag_operating","False","Achats d'emballages à usage unique et d'autres matériaux d'emballage et d'expédition"
"l10n_ca_511213","511213","Purchases of Goods for Resale (as is)","expense_direct_cost","account.account_tag_operating","False","Achats de biens pour la revente (tels quels)"
"l10n_ca_511220","511220","Costs Chargeable to Cost of Sales","expense_direct_cost","account.account_tag_operating","False","Coûts imputables au coût des marchandises vendues"
"l10n_ca_511300","511300","Closing Inventories","expense_direct_cost","account.account_tag_operating","False","Stocks de clôture"
"l10n_ca_511310","511310","Raw Materials and Components, Including Non-returnable Containers and Other Shipping and Packaging Materials","expense_direct_cost","account.account_tag_operating","False","Matières premières et composantes, y compris les emballages à usage unique et autres matériaux d'emballage et d'expédition"
"l10n_ca_511320","511320","Goods/Work in Process","expense_direct_cost","account.account_tag_operating","False","Biens en cours de fabrication ou travaux en cours"
"l10n_ca_511330","511330","Finished Goods","expense_direct_cost","account.account_tag_operating","False","Produits finis"
"l10n_ca_511331","511331","Goods Manufactured","expense_direct_cost","account.account_tag_operating","False","Produits fabriqués"
"l10n_ca_511332","511332","Goods Purchased for Resale (as is)","expense_direct_cost","account.account_tag_operating","False","Biens achetés pour la revente (tels quels)"
"l10n_ca_511340","511340","Other Inventories","expense_direct_cost","account.account_tag_operating","False","Autres stocks"
"l10n_ca_512100","512100","Labour Expenses","expense","account.account_tag_operating","False","Frais de main-d'oeuvre"
"l10n_ca_512110","512110","Salaries, Wages and Commissions","expense","account.account_tag_operating","False","Salaires, traitements et commissions"
"l10n_ca_512120","512120","Employee Benefits","expense","account.account_tag_operating","False","Avantages sociaux des employés"
"l10n_ca_512121","512121","Employee Current Benefits","expense","account.account_tag_operating","False","Avantages sociaux courants des employées"
"l10n_ca_512122","512122","Employee Future and Post-retirement Benefits","expense","account.account_tag_operating","False","Avantages futurs et postérieurs à la retraite des employés"
"l10n_ca_512200","512200","Expenses Other than Labour","expense","account.account_tag_operating","False","Charges autres que les frais de main-d'oeuvre"
"l10n_ca_512201","512201","Depreciation, Depletion and Amortization Expenses","expense_depreciation","account.account_tag_operating","False","Amortissement, y compris pour dépréciation et épuisement"
"l10n_ca_512202","512202","Energy (Including Vehicle Fuel), Water and Telephone/Telecommunication Expenses","expense","account.account_tag_operating","False","Frais d'énergie (carburant pour véhicules compris), d'eau, de téléphone et de télécommunications"
"l10n_ca_512203","512203","Indirect Taxes and Licences & Permits","expense","account.account_tag_operating","False","Impôts indirects, licences et permis"
"l10n_ca_512204","512204","Sub-contracts","expense","account.account_tag_operating","False","Contrats de sous-traitance"
"l10n_ca_512205","512205","Exploration and Development Expenses","expense","account.account_tag_operating","False","Frais d'exploration et d'aménagement"
"l10n_ca_512206","512206","Royalties and Franchise Fees Expenses","expense","account.account_tag_operating","False","Redevances de franchisage et autres redevances"
"l10n_ca_512207","512207","Purchased Materials, Supplies and Services","expense","account.account_tag_operating","False","Achats de matériaux, de fournitures et de services"
"l10n_ca_512210","512210","All Other Operating Expenses","expense","account.account_tag_operating","False","Autres frais d'exploitation"
"l10n_ca_521100","521100","Interest Expenses on Short-Term Debt","expense","","False","Intérêts débiteurs sur dettes à court terme"
"l10n_ca_521110","521110","Bank Loans","expense","","False","Prêts bancaires"
"l10n_ca_521120","521120","Commercial Paper","expense","","False","Effets de commerce"
"l10n_ca_521130","521130","Other Debt, short-term","expense","","False","Autres dettes, à court terme"
"l10n_ca_521200","521200","Interest Expenses on Long-Term Debt","expense","","False","Intérêts débiteurs sur dettes à long terme"
"l10n_ca_521210","521210","Mortgage Loans","expense","","False","Prêts hypothécaires"
"l10n_ca_521220","521220","Bond, Debentures","expense","","False","Obligations et débentures"
"l10n_ca_521230","521230","Other Long-term Debt","expense","","False","Autres dettes à long terme"
"l10n_ca_522000","522000","Other Non-operating Expenses","expense","","False","Autres frais non liés à l'exploitation"
"l10n_ca_522100","522100","Cash Discount Loss","expense","","False","Perte d'escompte"
"l10n_ca_522200","522200","Discounts Given","expense","","False","Remises accordées"
"l10n_ca_611000","611000","Gains/Losses on the Sale of Assets","income_other","","False","Gains ou pertes sur la vente d'actifs"
"l10n_ca_611100","611100","Realized Gains /Losses - Capital Assets","income_other","","False","Gains ou pertes réalisés - actifs immobilisés"
"l10n_ca_611200","611200","Realized Gains / Losses - Investments","income_other","","False","Gains ou pertes réalisés - placements"
"l10n_ca_611300","611300","Realized Gains / Losses - Other Assets","income_other","","False","Gains ou pertes réalisés - autres actifs"
"l10n_ca_612100","612100","Gains on the Translation of Foreign Currency","income_other","","False","Gains sur la conversion de devises"
"l10n_ca_612200","612200","Losses on the Translation of Foreign Currency","expense","","False","Pertes sur la conversion de devises"
"l10n_ca_613000","613000","Unrealized Losses on Asset Revaluations (Including Write-downs and Write-offs)","expense","","False","Pertes non réalisées sur la réévaluation d'actif (y compris réductions de valeur et radiations)"
"l10n_ca_613100","613100","Unrealized Losses - Capital Assets","expense","","False","Pertes non réalisées - actifs immobilisés"
"l10n_ca_613200","613200","Unrealized Losses - Investments","expense","","False","Pertes non réalisées - placements"
"l10n_ca_613300","613300","Unrealized Losses - Other Assets","expense","","False","Pertes non réalisées - autres actifs"
"l10n_ca_614000","614000","Gains on Settlement of Pension Obligation","income_other","","False","Gains sur règlement d'obligations au titre de régimes de retraite"
"l10n_ca_615000","615000","Gains/Losses Related to Litigation Settlement","income_other","","False","Gains ou pertes liés aux règlement de litiges"
"l10n_ca_616000","616000","Gains/Losses on Derivative Instruments (Including Hedge Revaluations)","income_other","","False","Gains ou pertes sur instruments dérivés (y compris les réévaluations d'opérations de couverture)"
"l10n_ca_621000","621000","Current Income Tax","expense","","False","Impôts exigibles de l'exercice"
"l10n_ca_622000","622000","Deferred Income Tax","expense","","False","Impôts reportés"
"l10n_ca_623000","623000","Provincial Mining and Logging Taxes","expense","","False","Impôt provincial sur l'exploitation minière et forestière"
"l10n_ca_631000","631000","Minority Shareholders' Portion of Net Income/Loss of Consolidated Subsidiaries and Affiliates","expense","","False","Part des bénéfices nets/pertes nettes des filiales et des sociétés affiliées consolidées revenant aux actionnaires minoritaires"
"l10n_ca_632100","632100","Equity in Net Income/Loss of Unconsolidated Subsidiaries and Affiliates","expense","","False","Quote-part des bénéfices nets ou pertes nettes des filiales et sociétés affiliées non consolidées"
"l10n_ca_632200","632200","Equity in Net Income/Loss of Partnerships and Joint Ventures","expense","","False","Quote-part des bénéfices nets ou pertes nettes des sociétés de personnes et coentreprises"
"l10n_ca_711000","711000","Extraordinary Items - Gross","expense","","False","Éléments extraordinaires - bruts"
"l10n_ca_712100","712100","Income Tax on Extraordinary Items - Current","expense","","False","Impôts sur les éléments extraordinaires - Exigibles"
"l10n_ca_712200","712200","Income Tax on Extraordinary Items - Deferred","expense","","False","Impôts sur les éléments extraordinaires - Reportés"
"l10n_ca_720000","720000","Non-recurring Items","expense","","False","Éléments non récurrents"
"l10n_ca_730000","730000","Adjustments due to Cumulative Effect of Changes in Accounting Principles","expense","","False","Redressements représentant l'effet cumulatif des changements de principes comptables"

```

## File: data\template\account.fiscal.position-ca_2023.csv

```csv
"id","name","auto_apply","country_id","state_ids","sequence","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id","name@fr"
"fiscal_position_template_ab","Alberta (AB)","1","base.ca","base.state_ca_ab","","gstpst_sale_tax_12_bc","gst_sale_tax_5","","","Alberta (AB)"
"","","","","","","gstpst_sale_tax_12_mb","gst_sale_tax_5","","",""
"","","","","","","hst_sale_tax_15","gst_sale_tax_5","","",""
"","","","","","","hst_sale_tax_13","gst_sale_tax_5","","",""
"","","","","","","gstqst_sale_tax_14975","gst_sale_tax_5","","",""
"","","","","","","gstpst_sale_tax_11","gst_sale_tax_5","","",""
"fiscal_position_template_bc","British Columbia (BC)","1","base.ca","base.state_ca_bc","","gstpst_sale_tax_12_mb","gstpst_sale_tax_12_bc","","","Colombie-Britannique (BC)"
"","","","","","","gst_sale_tax_5","gstpst_sale_tax_12_bc","","",""
"","","","","","","hst_sale_tax_15","gstpst_sale_tax_12_bc","","",""
"","","","","","","hst_sale_tax_13","gstpst_sale_tax_12_bc","","",""
"","","","","","","gstqst_sale_tax_14975","gstpst_sale_tax_12_bc","","",""
"","","","","","","gstpst_sale_tax_11","gstpst_sale_tax_12_bc","","",""
"fiscal_position_template_mb","Manitoba (MB)","1","base.ca","base.state_ca_mb","","gstpst_sale_tax_12_bc","gstpst_sale_tax_12_mb","","","Manitoba (MB)"
"","","","","","","gst_sale_tax_5","gstpst_sale_tax_12_mb","","",""
"","","","","","","hst_sale_tax_15","gstpst_sale_tax_12_mb","","",""
"","","","","","","hst_sale_tax_13","gstpst_sale_tax_12_mb","","",""
"","","","","","","gstqst_sale_tax_14975","gstpst_sale_tax_12_mb","","",""
"","","","","","","gstpst_sale_tax_11","gstpst_sale_tax_12_mb","","",""
"fiscal_position_template_nb","New Brunswick (NB)","1","base.ca","base.state_ca_nb","","gst_sale_tax_5","hst_sale_tax_15","","","Nouveau Brunswick (NB)"
"","","","","","","gstpst_sale_tax_12_bc","hst_sale_tax_15","","",""
"","","","","","","gstpst_sale_tax_12_mb","hst_sale_tax_15","","",""
"","","","","","","hst_sale_tax_13","hst_sale_tax_15","","",""
"","","","","","","gstqst_sale_tax_14975","hst_sale_tax_15","","",""
"","","","","","","gstpst_sale_tax_11","hst_sale_tax_15","","",""
"fiscal_position_template_nl","Newfoundland and Labrador (NL)","1","base.ca","base.state_ca_nl","","gst_sale_tax_5","hst_sale_tax_15","","","Terre-Neuve et Labrador (NL)"
"","","","","","","gstpst_sale_tax_12_bc","hst_sale_tax_15","","",""
"","","","","","","gstpst_sale_tax_12_mb","hst_sale_tax_15","","",""
"","","","","","","hst_sale_tax_13","hst_sale_tax_15","","",""
"","","","","","","gstqst_sale_tax_14975","hst_sale_tax_15","","",""
"","","","","","","gstpst_sale_tax_11","hst_sale_tax_15","","",""
"fiscal_position_template_ns","Nova Scotia (NS)","1","base.ca","base.state_ca_ns","","gst_sale_tax_5","hst_sale_tax_15","","","Nouvelle-Écosse (NS)"
"","","","","","","gstpst_sale_tax_12_bc","hst_sale_tax_15","","",""
"","","","","","","gstpst_sale_tax_12_mb","hst_sale_tax_15","","",""
"","","","","","","hst_sale_tax_13","hst_sale_tax_15","","",""
"","","","","","","gstqst_sale_tax_14975","hst_sale_tax_15","","",""
"","","","","","","gstpst_sale_tax_11","hst_sale_tax_15","","",""
"fiscal_position_template_nt","Northwest Territories (NT)","1","base.ca","base.state_ca_nt","","gstpst_sale_tax_12_bc","gst_sale_tax_5","","","Territoires du Nord-Ouest (NT)"
"","","","","","","gstpst_sale_tax_12_mb","gst_sale_tax_5","","",""
"","","","","","","hst_sale_tax_15","gst_sale_tax_5","","",""
"","","","","","","hst_sale_tax_13","gst_sale_tax_5","","",""
"","","","","","","gstqst_sale_tax_14975","gst_sale_tax_5","","",""
"","","","","","","gstpst_sale_tax_11","gst_sale_tax_5","","",""
"fiscal_position_template_nu","Nunavut (NU)","1","base.ca","base.state_ca_nu","","gstpst_sale_tax_12_bc","gst_sale_tax_5","","","Nunavut (NU)"
"","","","","","","gstpst_sale_tax_12_mb","gst_sale_tax_5","","",""
"","","","","","","hst_sale_tax_15","gst_sale_tax_5","","",""
"","","","","","","hst_sale_tax_13","gst_sale_tax_5","","",""
"","","","","","","gstqst_sale_tax_14975","gst_sale_tax_5","","",""
"","","","","","","gstpst_sale_tax_11","gst_sale_tax_5","","",""
"fiscal_position_template_on","Ontario (ON)","1","base.ca","base.state_ca_on","","gst_sale_tax_5","hst_sale_tax_13","","","Ontario (ON)"
"","","","","","","gstpst_sale_tax_12_bc","hst_sale_tax_13","","",""
"","","","","","","gstpst_sale_tax_12_mb","hst_sale_tax_13","","",""
"","","","","","","hst_sale_tax_15","hst_sale_tax_13","","",""
"","","","","","","gstqst_sale_tax_14975","hst_sale_tax_13","","",""
"","","","","","","gstpst_sale_tax_11","hst_sale_tax_13","","",""
"fiscal_position_template_pe","Prince Edward Islands (PE)","1","base.ca","base.state_ca_pe","","gst_sale_tax_5","hst_sale_tax_15","","","Îles du Prince-Édouard (PE)"
"","","","","","","gstpst_sale_tax_12_bc","hst_sale_tax_15","","",""
"","","","","","","gstpst_sale_tax_12_mb","hst_sale_tax_15","","",""
"","","","","","","hst_sale_tax_13","hst_sale_tax_15","","",""
"","","","","","","gstqst_sale_tax_14975","hst_sale_tax_15","","",""
"","","","","","","gstpst_sale_tax_11","hst_sale_tax_15","","",""
"fiscal_position_template_qc","Quebec (QC)","1","base.ca","base.state_ca_qc","","gstpst_sale_tax_12_bc","gstqst_sale_tax_14975","","","Québec (QC)"
"","","","","","","gstpst_sale_tax_12_mb","gstqst_sale_tax_14975","","",""
"","","","","","","hst_sale_tax_15","gstqst_sale_tax_14975","","",""
"","","","","","","hst_sale_tax_13","gstqst_sale_tax_14975","","",""
"","","","","","","gstpst_sale_tax_11","gstqst_sale_tax_14975","","",""
"","","","","","","gst_sale_tax_5","gstqst_sale_tax_14975","","",""
"fiscal_position_template_sk","Saskatchewan (SK)","1","base.ca","base.state_ca_sk","","gstpst_sale_tax_12_bc","gstpst_sale_tax_11","","","Saskatchewan (SK)"
"","","","","","","gst_sale_tax_5","gstpst_sale_tax_11","","",""
"","","","","","","gstpst_sale_tax_12_mb","gstpst_sale_tax_11","","",""
"","","","","","","hst_sale_tax_15","gstpst_sale_tax_11","","",""
"","","","","","","hst_sale_tax_13","gstpst_sale_tax_11","","",""
"","","","","","","gstqst_sale_tax_14975","gstpst_sale_tax_11","","",""
"fiscal_position_template_yt","Yukon (YT)","1","base.ca","base.state_ca_yt","","gstpst_sale_tax_12_bc","gst_sale_tax_5","","","Yukon (YT)"
"","","","","","","gstpst_sale_tax_12_mb","gst_sale_tax_5","","",""
"","","","","","","hst_sale_tax_15","gst_sale_tax_5","","",""
"","","","","","","hst_sale_tax_13","gst_sale_tax_5","","",""
"","","","","","","gstqst_sale_tax_14975","gst_sale_tax_5","","",""
"","","","","","","gstpst_sale_tax_11","gst_sale_tax_5","","",""
"fiscal_position_template_intl","International (INTL)","1","","","","gst_sale_tax_5","int_sale_tax_0","","","International (INTL)"
"","","","","","","gstpst_sale_tax_11","int_sale_tax_0","","",""
"","","","","","","gstpst_sale_tax_12_bc","int_sale_tax_0","","",""
"","","","","","","gstpst_sale_tax_12_mb","int_sale_tax_0","","",""
"","","","","","","gstqst_sale_tax_14975","int_sale_tax_0","","",""
"","","","","","","hst_sale_tax_13","int_sale_tax_0","","",""
"","","","","","","hst_sale_tax_15","int_sale_tax_0","","",""

```

## File: data\template\account.group-ca_2023.csv

```csv
"id","code_prefix_start","name","name@fr"
"l10n_ca_group_1","1","Assets","Actif"
"l10n_ca_group_11","11","Financial Assets","Actifs financiers"
"l10n_ca_group_111","111","Cash, Deposits and Official International Reserves","Encaisse, dépôts et réserves internationales officielles"
"l10n_ca_group_112","112","Accounts Receivable - Net","Comptes débiteurs - Net"
"l10n_ca_group_1121","1121","Accounts Receivable and Accrued Revenue - Gross","Comptes débiteurs et revenus courus - Brut"
"l10n_ca_group_11211","11211","Trade Accounts Receivable","Comptes clients"
"l10n_ca_group_11212","11212","Rent and Operating Lease Receivable","Comptes débiteurs - location et location-exploitation"
"l10n_ca_group_11213","11213","Accrued Interest Receivable and other Accrued Investment Income (non-affiliates)","Intérêts courus à recevoir et autres revenus de placements à recevoir (sociétés non affiliées)"
"l10n_ca_group_11214","11214","Finance Receivable (non-affiliates)","Effets financiers à recevoir (sociétés non affiliées)"
"l10n_ca_group_11215","11215","Notes Receivable (non-affiliates)","Billets à recevoir (sociétés non affiliées)"
"l10n_ca_group_11216","11216","Government assistance receivable","Aide gouvernementale à recevoir"
"l10n_ca_group_1122","1122","Allowance for Doubtful Accounts","Provision pour créances douteuses"
"l10n_ca_group_113","113","Investments in Affiliates","Placements dans des sociétés affiliées"
"l10n_ca_group_1131","1131","Investments in Canadian Affiliates, Equity Investment in Corporations, including cost of shares","Placements dans des sociétés affiliées canadiennes, placement en actions dans des sociétés, y compris le coût des actions"
"l10n_ca_group_1132","1132","Investments in Foreign Affiliates, Equity investment in Corporations, including cost of shares","Placements dans des sociétés affiliées étrangères, placement en actions dans des sociétés, y compris le coût des actions"
"l10n_ca_group_1133","1133","Loans, Notes, Accounts, Bonds, Mortgages and Other Investments","Prêts, billets, comptes débiteurs, obligations, hypothèques et autres placements"
"l10n_ca_group_114","114","Investments in Non-Affiliates (Portfolio Securities)","Placements dans des entreprises non affiliées (titres de portefeuille)"
"l10n_ca_group_1141","1141","Investments in Canadian Debt Instruments (Non-Affiliated)","Placements en titres d'emprunt canadiens (sociétés non affiliées)"
"l10n_ca_group_1142","1142","Investments in Canadian Equity Instruments (Non-Affiliated)","Placements en instruments de capitaux propres canadiens (sociétés non affiliées)"
"l10n_ca_group_1143","1143","Investments in Foreign Non-Affiliates","Placements dans des sociétés non affiliées étrangères"
"l10n_ca_group_115","115","Mortgage Loans to Non-Affiliates","Prêts hypothécaires aux sociétés non affiliées"
"l10n_ca_group_116","116","Non-Mortgage Loans to Non-Affiliates","Prêts non hypothécaires aux sociétés non affiliées"
"l10n_ca_group_1161","1161","Non-Mortgage Loans made by Enterprises in Non-Financial Industries","Prêts non hypothécaires consentis par des entreprises non financières"
"l10n_ca_group_1162","1162","Allowance for Losses on Non-Mortgage Loans","Provision pour pertes sur prêts non hypothécaires"
"l10n_ca_group_118","118","Tax Receivables","Taxes recevable"
"l10n_ca_group_12","12","Tangible Assets","Actifs corporels"
"l10n_ca_group_121","121","Inventories","Stocks"
"l10n_ca_group_1211","1211","Goods Inventories","Stocks de biens"
"l10n_ca_group_12111","12111","Manufacturing Inventories","Stocks manufacturiers"
"l10n_ca_group_12112","12112","Goods for Sale Inventory","Stock de biens destinés à la vente"
"l10n_ca_group_1212","1212","Part and Supplies Inventories","Stocks de pièces et fournitures"
"l10n_ca_group_1213","1213","Real Estate Held or being Developed for Sale","Biens immobiliers détenus ou en cours d'aménagement en vue de la vente"
"l10n_ca_group_122","122","Allowance for Obsolescence and decline in Value","Provision pour obsolescence et dévaluation"
"l10n_ca_group_123","123","Breeding Livestock","Animaux de reproduction"
"l10n_ca_group_124","124","Capital Assets","Actifs immobilisés"
"l10n_ca_group_1241","1241","Fixed Assets - Net","Immobilisations corporelles - Nets"
"l10n_ca_group_1242","1242","Depletable Assets - Net","Actifs non renouvelables - nets"
"l10n_ca_group_1243","1243","Repossessed Assets Held for Sale","Reprise de possession d'immobilisations destinées à la vente"
"l10n_ca_group_13","13","Deferred Charges and Intangible Assets","Frais reportés et actifs incorporels"
"l10n_ca_group_131","131","Deferred Charges and Prepaid Expenses","Frais reportés et charges payées d'avance"
"l10n_ca_group_1311","1311","Deferred Charges - Net","Frais reportés - nets"
"l10n_ca_group_1312","1312","Prepaid Expenses","Frais payés d'avance"
"l10n_ca_group_132","132","Intangible Assets - Net","Actifs incorporels - nets"
"l10n_ca_group_1321","1321","Goodwill - Net","Achalandage - net"
"l10n_ca_group_1322","1322","Software","Logiciels"
"l10n_ca_group_1323","1323","Database","Bases de données"
"l10n_ca_group_1324","1324","Excess of Purchase Price of Consolidated Subsidiary Shares","Excédent du prix d'achat d'actions d'une filiale consolidée"
"l10n_ca_group_1325","1325","Patents, Trademarks and Copyrights - Net","Brevets, marques de commerce et droits d'auteurs - nets"
"l10n_ca_group_1326","1326","Other Intangibles","Autres actifs incorporels"
"l10n_ca_group_13261","13261","Quota - Net","Contingents - nets"
"l10n_ca_group_13262","13262","License - Net","Permis - nets"
"l10n_ca_group_13263","13263","Incorporation Cost - Net","Frais de constitution - nets"
"l10n_ca_group_13264","13264","Customer List - Net","Liste de clients - nets"
"l10n_ca_group_13265","13265","Right - Net","Droits - nets"
"l10n_ca_group_13266","13266","Timber Rights - Net","Droits de coupe - nets"
"l10n_ca_group_13267","13267","Mining Right - Net","Droits miniers - nets"
"l10n_ca_group_13268","13268","Oil and Gas Right - Net","Droits pétroliers et gaziers - nets"
"l10n_ca_group_13269","13269","All Other Intangibles Assets - Net","Tous autres actifs incorporels - nets"
"l10n_ca_group_14","14","Other Assets","Autres éléments d'actif"
"l10n_ca_group_141","141","Pensions","Régimes de retraite"
"l10n_ca_group_142","142","All Other Assets","Tout autre élément d'actif"
"l10n_ca_group_1421","1421","Other Deposits","Autres dépôts"
"l10n_ca_group_14213","14213","Deposits with Others","Dépôts chez d'autres entités"
"l10n_ca_group_1422","1422","All Other Assets, except Other Deposits","Tous autres actifs, sauf autres dépôts"
"l10n_ca_group_14221","14221","Reserve Fund","Fonds de réserve"
"l10n_ca_group_14222","14222","Cash Surrender Value of Life Insurance","Valeur de rachat de l'assurance vie"
"l10n_ca_group_14223","14223","Asset Held in Trust","Éléments d'actif détenus en fiducie"
"l10n_ca_group_14224","14224","Other","Autre"
"l10n_ca_group_2","2","Liabilities","Passif"
"l10n_ca_group_21","21","Financial Instruments and Borrowing","Instruments financiers et emprunts"
"l10n_ca_group_211","211","Mortgage Loans","Prêts hypothécaires"
"l10n_ca_group_2111","2111","Residential Mortgage Loans","Prêts hypothécaires résidentiels"
"l10n_ca_group_2112","2112","Non-Residential Mortgage Loans","Prêts hypothécaires non résidentiels"
"l10n_ca_group_212","212","Non-Mortgage Loans","Prêts non hypothécaires"
"l10n_ca_group_2121","2121","Non-Mortgage Loans, Bank Loans - Overdrafts and Loans from Chartered Banks and Branches of Foreign Banks Operating in Canada (excluding Lien Notes Payable)","Prêts non hypothécaires et prêts de banque - Découverts et prêts auprès de banques à charte et de succursales de banques étrangères établies au Canada (à l'exclusion des billets portant privilège)"
"l10n_ca_group_2122","2122","Non-Mortgage Loans, Loans from Other Domestic Lending Institutions","Prêts non hypothécaires, prêts auprès d'autres sociétés de financement canadiennes"
"l10n_ca_group_2123","2123","Non-Mortgage Loans, Loans from Others","Prêts non hypothécaires, prêts auprès d'autres entités"
"l10n_ca_group_2124","2124","Non-Mortgage Loans, Lien Note Payable","Prêts non hypothécaires, billets portant privilège à payer"
"l10n_ca_group_2125","2125","Obligations under Financial/Capital Leases (excluding from Chartered Banks)","Obligations (autres qu'envers les banques à charte) - contrats de location-acquisition ou crédit-bail"
"l10n_ca_group_213","213","Debt Securities","Titres de créance"
"l10n_ca_group_2131","2131","Debt Securities, Short-term Paper, Commercial Paper, Finance Company Paper and Bankers' Acceptances","Titres de créance, effets à court terme, effets de commerce, effets de société de financement et acceptations bancaires"
"l10n_ca_group_2132","2132","Debt Securities, Bonds, Debentures and Promissory Notes","Titres de créance, obligations, débentures et billets"
"l10n_ca_group_214","214","Derivative Liabilities (Excluding Employee Stock Options)","Passifs dérivés (sauf options d'achat d'actions accordées à des employés)"
"l10n_ca_group_215","215","Current portion of long-term debt","Partie à court terme de la dette à long terme"
"l10n_ca_group_22","22","Claims","Créances"
"l10n_ca_group_221","221","Accounts Payable and Accrued Liabilities","Comptes créditeurs et charges à payer"
"l10n_ca_group_2211","2211","Accounts Payable, Trade","Comptes créditeurs, commerce"
"l10n_ca_group_2212","2212","Other Accounts Payable and Accruals (Non-Affiliates)","Autres comptes créditeurs et charges à payer (sociétés non affiliées)"
"l10n_ca_group_222","222","Claims of Affiliates","Créances de sociétés affiliées"
"l10n_ca_group_23","23","Taxes collected","Taxes collectées"
"l10n_ca_group_24","24","Future Income Taxes","Passifs d'impôts futurs"
"l10n_ca_group_25","25","Minority Interest in Subsidiaries Consolidated","Intérêts minoritaires dans les filiales consolidées"
"l10n_ca_group_26","26","Pension Plans and Deferred Profit Sharing Plans","Régimes de retraite et régimes de participation différée aux bénéfices"
"l10n_ca_group_27","27","Provisions for Future Obligations","Provision pour obligations futures"
"l10n_ca_group_28","28","Customer Deposits, Deferred income (including Deferred Revenue from Incomplete Contracts)","Dépôts de clients, revenus reportés (y compris les revenus reportés de contrats incomplets)"
"l10n_ca_group_29","29","Other Liabilities","Autres éléments de passif"
"l10n_ca_group_291","291","Deferred Gains/Losses on Translation of Foreign Currency","Gains ou pertes reportés sur conversion de devises"
"l10n_ca_group_292","292","Contributions to Environmental Trust","Contributions à une fiducie pour l'environnement"
"l10n_ca_group_293","293","Restated Preference Shares","Actions privilégiées ajustées"
"l10n_ca_group_294","294","Amounts Held in Trust","Montants détenus en fiducie"
"l10n_ca_group_295","295","Stock Appreciation Rights - settled in cash","Droits à la plus-value d'actions - réglés en espèces"
"l10n_ca_group_296","296","Liabilities of disposal groups classified as held for sale","Passifs des groupes à céder classés comme détenus en vue de la vente"
"l10n_ca_group_297","297","All Other non-current Liabilities","Tous les autres éléments de passif non-courants"
"l10n_ca_group_298","298","All Other current Liabilities","Tous les autres éléments de passif courants"
"l10n_ca_group_3","3","Equity","Capitaux propres"
"l10n_ca_group_31","31","Share Capital","Capital-actions"
"l10n_ca_group_311","311","Preferred Shares","Actions privilégiées"
"l10n_ca_group_312","312","Common Shares","Actions ordinaires"
"l10n_ca_group_313","313","Owners' Equity (Owners' Investment - No Share Capital Entities)","Avoir propre (mise de fonds du propriétaire - entités sans capital-actions)"
"l10n_ca_group_314","314","Other Shares","Autres actions"
"l10n_ca_group_32","32","Retained Earnings/Deficits","Bénéfices non repartis / déficits"
"l10n_ca_group_33","33","Contributed Surplus","Surplus d'apport"
"l10n_ca_group_34","34","Other Surplus","Autres surplus"
"l10n_ca_group_35","35","Non-controlling interests","Intérêts minoritaires"
"l10n_ca_group_36","36","Reserves","Réserves"
"l10n_ca_group_4","4","Revenue","Revenus"
"l10n_ca_group_41","41","Operating Revenue","Revenus d'exploitation"
"l10n_ca_group_411","411","Sales of Goods and Services","Ventes de biens et de services"
"l10n_ca_group_4111","4111","Sales of Goods","Ventes de biens"
"l10n_ca_group_4112","4112","Sales of Services","Ventes de services"
"l10n_ca_group_413","413","Other Operating Revenue","Autres revenus d'exploitation"
"l10n_ca_group_4133","4133","All Other Operating Revenue","Autres revenus d'exploitation"
"l10n_ca_group_42","42","Non-Operating Revenue","Revenus non liés à l'exploitation"
"l10n_ca_group_421","421","Dividends","Dividendes"
"l10n_ca_group_422","422","Interest","Intérêts"
"l10n_ca_group_423","423","Other Non-operating Revenue","Autres revenus non liés à l'exploitation"
"l10n_ca_group_5","5","Expenses","Dépenses"
"l10n_ca_group_51","51","Operating Expenses (Including Cost of Goods Sold)","Dépenses d'exploitation (y compris le coût des biens vendus)"
"l10n_ca_group_511","511","Cost of Goods Sold","Coût des biens vendus"
"l10n_ca_group_5111","5111","Opening Inventories","Stocks d'ouverture"
"l10n_ca_group_51111","51111","Raw Materials and Components, Including Non-returnable Containers and Other Shipping and Packaging Materials","Matières premières et composantes, y compris les emballages à usage unique et autres matériaux d'emballage et d'expédition"
"l10n_ca_group_51112","51112","Goods/Work in Process","Biens en cours de fabrication ou travaux en cours"
"l10n_ca_group_51113","51113","Finished Goods","Produits finis"
"l10n_ca_group_51114","51114","Other Inventories","Autres stocks"
"l10n_ca_group_5112","5112","Purchases and Costs Chargeable to Cost of Sales","Achats et coûts imputables au coût des marchandises vendues"
"l10n_ca_group_51121","51121","Purchases","Achats"
"l10n_ca_group_51122","51122","Costs Chargeable to Cost of Sales","Coûts imputables au coût des marchandises vendues"
"l10n_ca_group_5113","5113","Closing Inventories","Stocks de clôture"
"l10n_ca_group_51131","51131","Raw Materials and Components, Including Non-returnable Containers and Other Shipping and Packaging Materials","Matières premières et composantes, y compris les emballages à usage unique et autres matériaux d'emballage et d'expédition"
"l10n_ca_group_51132","51132","Goods/Work in Process","Biens en cours de fabrication ou travaux en cours"
"l10n_ca_group_51133","51133","Finished Goods","Produits finis"
"l10n_ca_group_51134","51134","Other Inventories","Autres stocks"
"l10n_ca_group_512","512","Operating Expenses Not Included in Cost of Goods Sold","Frais d'exploitation non compris dans le coût des biens vendus"
"l10n_ca_group_5121","5121","Labour Expenses","Frais de main-d'oeuvre"
"l10n_ca_group_5122","5122","Expenses Other than Labour","Charges autres que les frais de main-d'oeuvre"
"l10n_ca_group_52","52","Non-Operating Expenses","Frais non liés à l'exploitation"
"l10n_ca_group_521","521","Interest Expenses","Intérêts débiteurs"
"l10n_ca_group_5211","5211","Interest Expenses on Short-Term Debt","Intérêts débiteurs sur dettes à court terme"
"l10n_ca_group_5212","5212","Interest Expenses on Long-Term Debt","Intérêts débiteurs sur dettes à long terme"
"l10n_ca_group_522","522","Other Non-operating Expenses","Autres frais non liés à l'exploitation"
"l10n_ca_group_6","6","Gains/Losses, Corporate Taxes and Other Items","Gains ou pertes, impôts des sociétés et autres éléments"
"l10n_ca_group_61","61","Gains/Losses","Gains ou pertes"
"l10n_ca_group_611","611","Gains/Losses on the Sale of Assets","Gains ou pertes sur la vente d'actifs"
"l10n_ca_group_612","612","Gains/Losses on the Translation of Foreign Currency","Gains ou pertes sur la conversion de devises"
"l10n_ca_group_613","613","Unrealized Losses on Asset Revaluations (Including Write-downs and Write-offs)","Pertes non réalisées sur la réévaluation d'actif (y compris réductions de valeur et radiations)"
"l10n_ca_group_614","614","Gains on Settlement of Pension Obligation","Gains sur règlement d'obligations au titre de régimes de retraite"
"l10n_ca_group_615","615","Gains/Losses Related to Litigation Settlement","Gains ou pertes liés aux règlement de litiges"
"l10n_ca_group_616","616","Gains/Losses on Derivative Instruments (Including Hedge Revaluations)","Gains ou pertes sur instruments dérivés (y compris les réévaluations d'opérations de couverture)"
"l10n_ca_group_62","62","Corporate Income Tax","Impôts sur les bénéfices des sociétés"
"l10n_ca_group_63","63","Other Items","Autres éléments"
"l10n_ca_group_631","631","Minority Shareholders' Portion of Net Income/Loss of Consolidated Subsidiaries and Affiliates","Part des bénéfices nets/pertes nettes des filiales et des sociétés affiliées consolidées revenant aux actionnaires minoritaires"
"l10n_ca_group_632","632","Net Income/Loss of Unconsolidated Subsidiaries, Affiliates, Partnerships and Joint Ventures (Equity Method)","Bénéfices nets ou pertes nettes des filiales, sociétés affiliées, sociétés de personnes et coentreprises non consolidées (selon la méthode de la comptabilisation à la valeur de consolidation)"
"l10n_ca_group_7","7","Extraordinary Gains/Losses, Non-recurring Items & Adjustments","Gains ou pertes extraordinaires, éléments non récurrents et redressements"
"l10n_ca_group_71","71","Extraordinary Gains/Losses","Gains ou pertes extraordinaires"
"l10n_ca_group_711","711","Extraordinary Items - Gross","Éléments extraordinaires - bruts"
"l10n_ca_group_712","712","Income Tax on Extraordinary Items","Impôts sur les éléments extraordinaires"
"l10n_ca_group_72","72","Non-recurring Items","Éléments non récurrents"
"l10n_ca_group_73","73","Adjustments due to Cumulative Effect of Changes in Accounting Principles","Redressements représentant l'effet cumulatif des changements de principes comptables"

```

## File: data\template\account.tax-ca_2023.csv

```csv
"id","name","invoice_label","description","type_tax_use","amount","amount_type","sequence","include_base_amount","tax_group_id","children_tax_ids","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@fr","name@fr"
"gst_sale_tax_5","5% GST","5%","5% GST","sale","5.0","percent","10","False","tax_group_gst","","base","invoice","+Ca90||+CaBcA||+CaQc101","","5% TPS","5% TPS"
"","","","","","","","","","","","tax","invoice","+Ca103||+CaQc103","l10n_ca_231000","",""
"","","","","","","","","","","","base","refund","-Ca90||-CaBcA||-CaQc101","","",""
"","","","","","","","","","","","tax","refund","-Ca103||-CaQc103","l10n_ca_231000","",""
"gst_sale_tax_0","0% GST","0%","0% GST","sale","0.0","percent","10","False","tax_group_gst","","base","invoice","+Ca90||+CaBcA||+CaQc101","","0% TPS","0% TPS"
"","","","","","","","","","","","tax","invoice","+Ca103||+CaQc103","l10n_ca_231000","",""
"","","","","","","","","","","","base","refund","-Ca90||-CaBcA||-CaQc101","","",""
"","","","","","","","","","","","tax","refund","-Ca103||-CaQc103","l10n_ca_231000","",""
"pst_sale_tax_6","6% PST SK","6%","6% PST SK","sale","6.0","percent","20","False","tax_group_pst_6","","base","invoice","+CaSk8Base","","6% TVP SK","6% TVP SK"
"","","","","","","","","","","","tax","invoice","+CaSk8","l10n_ca_232000","",""
"","","","","","","","","","","","base","refund","-CaSk8Base","","",""
"","","","","","","","","","","","tax","refund","+CaSk3","l10n_ca_232000","",""
"pst_sale_tax_7_bc","7% PST BC","7%","7% PST BC","sale","7.0","percent","20","False","tax_group_pst_7","","base","invoice","","","7% TVP BC","7% TVP BC"
"","","","","","","","","","","","tax","invoice","+CaBcB","l10n_ca_232000","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","+CaBcI","l10n_ca_232000","",""
"pst_sale_tax_7_mb","7% PST MB","7%","7% PST MB","sale","7.0","percent","20","False","tax_group_pst_7","","base","invoice","","","7% TVP MB","7% TVP MB"
"","","","","","","","","","","","tax","invoice","+CaMb1","l10n_ca_232000","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-CaMb1","l10n_ca_232000","",""
"qst_sale_tax_9975","9.975% QST","9.975%","9.975% QST","sale","9.975","percent","20","False","tax_group_qst","","base","invoice","","","9.975% TVQ","9.975% TVQ"
"","","","","","","","","","","","tax","invoice","+CaQc203","l10n_ca_232000","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-CaQc203","l10n_ca_232000","",""
"hst_sale_tax_13","13% HST","13%","13% HST","sale","13.0","percent","20","False","tax_group_hst_13","","base","invoice","+Ca90||+CaBcA||+CaQc101","","13% TVH","13% TVH"
"","","","","","","","","","","","tax","invoice","+Ca103||+CaQc103","l10n_ca_233000","",""
"","","","","","","","","","","","base","refund","-Ca90||-CaBcA||-CaQc101","","",""
"","","","","","","","","","","","tax","refund","-Ca103||-CaQc103","l10n_ca_233000","",""
"hst_sale_tax_15","15% HST","15%","15% HST","sale","15.0","percent","20","False","tax_group_hst_15","","base","invoice","+Ca90||+CaBcA||+CaQc101","","15% TVH","15% TVH"
"","","","","","","","","","","","tax","invoice","+Ca103||+CaQc103","l10n_ca_235000","",""
"","","","","","","","","","","","base","refund","-Ca90||-CaBcA||-CaQc101","","",""
"","","","","","","","","","","","tax","refund","-Ca103||-CaQc103","l10n_ca_235000","",""
"gstpst_sale_tax_11","11% GST+PST SK","11%","5% GST + 6% PST SK","sale","100.0","group","","","tax_group_fix","gst_sale_tax_5,pst_sale_tax_6","","","","","5% TPS + 6% TVP SK","11% TPS+TVP SK"
"gstpst_sale_tax_12_bc","12% GST+PST BC","12%","5% GST + 7% PST BC","sale","100.0","group","","","tax_group_fix","gst_sale_tax_5,pst_sale_tax_7_bc","","","","","5% TPS + 7% TVP BC","12% TPS+TVP BC"
"gstpst_sale_tax_12_mb","12% GST+PST MB","12%","5% GST + 7% PST MB","sale","100.0","group","","","tax_group_fix","gst_sale_tax_5,pst_sale_tax_7_mb","","","","","5% TPS + 7% TVP MB","12% TPS+TVP MB"
"gstqst_sale_tax_14975","14.975% GST+QST","14.975%","5% GST + 9.975% QST","sale","100.0","group","","","tax_group_fix","gst_sale_tax_5,qst_sale_tax_9975","","","","","5% TPS + 9.975% TVQ","14.975% TPS+TVQ"
"int_sale_tax_0","0% Int","0%","0% International","sale","0.0","percent","10","False","tax_group_fix","","base","invoice","+Ca91","","0% International","0% Int"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-Ca91","","",""
"","","","","","","","","","","","tax","refund","","","",""
"exempt_sale_tax_0","0% Exempt","0%","0% Exempt","sale","0.0","percent","10","False","tax_group_fix","","base","invoice","+Ca91","","0% TPS","0% TPS"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","-Ca91","","",""
"","","","","","","","","","","","tax","refund","","","",""
"gst_purchase_tax_5","5% GST","5%","5% GST","purchase","5.0","percent","10","False","tax_group_gst","","base","invoice","","","5% TPS","5% TPS"
"","","","","","","","","","","","tax","invoice","+Ca106||+CaQc106","l10n_ca_118100","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-Ca106||-CaQc106","l10n_ca_118100","",""
"gst_purchase_tax_0","0% GST","0%","0% GST","purchase","0.0","percent","10","False","tax_group_gst","","base","invoice","","","0% TPS","0% TPS"
"","","","","","","","","","","","tax","invoice","+Ca106||+CaQc106","l10n_ca_118100","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-Ca106||-CaQc106","l10n_ca_118100","",""
"pst_purchase_tax_6","6% PST SK","6%","6% PST SK","purchase","6.0","percent","20","False","tax_group_pst_6","","base","invoice","","","6% TVP SK","6% TVP SK"
"","","","","","","","","","","","tax","invoice","","l10n_ca_118200","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","","l10n_ca_118200","",""
"pst_purchase_tax_6_G_resale","6% PST SK R","6%","6% PST SK Goods for resale","purchase","6.0","percent","20","False","tax_group_pst_6","","base","invoice","","","6% TVP SK Biens destinés à la revente","6% TVP SK B R"
"","","","","","","","","","","","tax","invoice","+CaSk2","l10n_ca_118200","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-CaSk2","l10n_ca_118200","",""
"pst_purchase_tax_7_bc","7% PST BC","7%","7% PST BC","purchase","7.0","percent","20","False","tax_group_pst_7","","base","invoice","","","7% TVP BC","7% TVP BC"
"","","","","","","","","","","","tax","invoice","+CaBcF","l10n_ca_118200","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-CaBcF","l10n_ca_118200","",""
"pst_purchase_tax_7_mb","7% PST MB","7%","7% PST MB","purchase","7.0","percent","20","False","tax_group_pst_7","","base","invoice","","","7% TVP MB","7% TVP MB"
"","","","","","","","","","","","tax","invoice","","l10n_ca_118200","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","","l10n_ca_118200","",""
"qst_purchase_tax_9975","9.975% QST","9.975%","9.975% QST","purchase","9.975","percent","20","False","tax_group_qst","","base","invoice","","","9.975% TVQ","9.975% TVQ"
"","","","","","","","","","","","tax","invoice","+CaQc206","l10n_ca_118200","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-CaQc206","l10n_ca_118200","",""
"hst_purchase_tax_13","13% HST","13%","13% HST","purchase","13.0","percent","20","False","tax_group_hst_13","","base","invoice","","","13% TVH","13% TVH"
"","","","","","","","","","","","tax","invoice","+Ca106||+CaQc106","l10n_ca_118300","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-Ca106||-CaQc106","l10n_ca_118300","",""
"hst_purchase_tax_15","15% HST","15%","15% HST","purchase","15.0","percent","20","False","tax_group_hst_15","","base","invoice","","","15% TVH","15% TVH"
"","","","","","","","","","","","tax","invoice","+Ca106||+CaQc106","l10n_ca_118500","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-Ca106||-CaQc106","l10n_ca_118500","",""
"gstpst_purchase_tax_11","11% GST+PST SK","11%","5% GST + 6% PST SK","purchase","100.0","group","","","tax_group_fix","gst_purchase_tax_5,pst_purchase_tax_6","","","","","5% TPS + 6% TVP SK","11% TPS+TVP SK"
"gstpst_purchase_tax_11_G_resale","11% GST+PST SK G R","11%","5% GST + 6% PST SK Goods for resale","purchase","100.0","group","","","tax_group_fix","gst_purchase_tax_5,pst_purchase_tax_6_G_resale","","","","","5% TPS + 6% TVP SK Biens destinés à la revente","11% TPS+TVP SK B R"
"gstpst_purchase_tax_12_bc","12% GST+PST BC","12%","5% GST + 7% PST BC","purchase","100.0","group","","","tax_group_fix","gst_purchase_tax_5,pst_purchase_tax_7_bc","","","","","5% TPS + 7% TVP BC","12% TPS+TVP BC"
"gstpst_purchase_tax_12_mb","12% GST+PST MB","12%","5% GST + 7% PST MB","purchase","100.0","group","","","tax_group_fix","gst_purchase_tax_5,pst_purchase_tax_7_mb","","","","","5% TPS + 7% TVP MB","12% TPS+TVP MB"
"gstqst_purchase_tax_14975","14.975% GST+QST","14.975%","5% GST + 9.975% QST","purchase","100.0","group","","","tax_group_fix","gst_purchase_tax_5,qst_purchase_tax_9975","","","","","5% TPS + 9.975% TVQ","14.975% TPS+TVQ"
"int_purchase_tax_0","0% Int","0%","0% International","purchase","0.0","percent","10","False","tax_group_fix","","base","invoice","","","0% International","0% Int"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","","","",""
"exempt_purchase_tax_0","0% Exempt","0%","0% Exempt","purchase","0.0","percent","10","False","tax_group_fix","","base","invoice","","","0% TPS","0% TPS"
"","","","","","","","","","","","tax","invoice","","","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","","","",""

```

## File: data\template\account.tax.group-ca_2023.csv

```csv
"id","name","country_id","name@fr"
"tax_group_fix","Taxes","base.ca",""
"tax_group_gst","GST","base.ca","TPS"
"tax_group_pst_6","PST 6%","base.ca","TVP 6%"
"tax_group_pst_7","PST 7%","base.ca","TVP 7%"
"tax_group_qst","QST","base.ca","TVQ"
"tax_group_hst_13","HST 13%","base.ca","TVH 13%"
"tax_group_hst_15","HST 15%","base.ca","TVH 15%"

```

## File: models\res_company.py

```python
from odoo import models, fields


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_ca_pst = fields.Char(related='partner_id.l10n_ca_pst', string='PST Number', store=False, readonly=False)


class BaseDocumentLayout(models.TransientModel):
    _inherit = 'base.document.layout'

    l10n_ca_pst = fields.Char(related='company_id.l10n_ca_pst', readonly=True)
    account_fiscal_country_id = fields.Many2one(related="company_id.account_fiscal_country_id", readonly=True)

```

## File: models\res_partner.py

```python
from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_ca_pst = fields.Char(string='PST number', help='Canadian Provincial Tax Identification Number')

```

## File: models\template_ca.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ca_2023')
    def _get_ca_template_data(self):
        return {
            'property_account_receivable_id': 'l10n_ca_112110',
            'property_account_payable_id': 'l10n_ca_221110',
            'property_account_income_categ_id': 'l10n_ca_411100',
            'property_account_expense_categ_id': 'l10n_ca_511210',
            'property_stock_account_input_categ_id': 'l10n_ca_121130',
            'property_stock_account_output_categ_id': 'l10n_ca_121140',
            'property_stock_valuation_account_id': 'l10n_ca_121120',
        }

    @template('ca_2023', 'res.company')
    def _get_ca_res_company(self):

        default_sales_tax, default_purchase_tax = {
            'BC': ('gstpst_sale_tax_12_bc', 'gstpst_purchase_tax_12_bc'),
            'MB': ('gstpst_sale_tax_12_mb', 'gstpst_purchase_tax_12_mb'),
            'QC': ('gstqst_sale_tax_14975', 'gstqst_purchase_tax_14975'),
            'SK': ('gstpst_sale_tax_11', 'gstpst_purchase_tax_11'),
            'ON': ('hst_sale_tax_13', 'hst_purchase_tax_13'),
            'NB': ('hst_sale_tax_15', 'hst_purchase_tax_15'),
            'NL': ('hst_sale_tax_15', 'hst_purchase_tax_15'),
            'NS': ('hst_sale_tax_15', 'hst_purchase_tax_15'),
            'PE': ('hst_sale_tax_15', 'hst_purchase_tax_15'),
        }.get(self.env.company.state_id.code, ('gst_sale_tax_5', 'gst_purchase_tax_5'))

        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.ca',
                'bank_account_code_prefix': '11131',
                'cash_account_code_prefix': '11121',
                'transfer_account_code_prefix': '1111',
                'account_default_pos_receivable_account_id': 'l10n_ca_112113',
                'income_currency_exchange_account_id': 'l10n_ca_423100',
                'expense_currency_exchange_account_id': 'l10n_ca_522100',
                'account_journal_early_pay_discount_loss_account_id': 'l10n_ca_522200',
                'account_journal_early_pay_discount_gain_account_id': 'l10n_ca_423200',
                'account_sale_tax_id': default_sales_tax,
                'account_purchase_tax_id': default_purchase_tax,
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_ca
from . import res_partner
from . import res_company

```

## File: views\report_invoice.xml

```xml
<odoo>
    <template id="l10n_ca_report_invoice_document_inherit" inherit_id="account.report_invoice_document">
        <xpath expr="//div[hasclass('row')]//div[@name='address_not_same_as_shipping']//t[@t-set='address']" position="inside">
            <div t-if="o.partner_id.l10n_ca_pst">PST: <span t-field="o.partner_id.l10n_ca_pst"/></div>
        </xpath>
        <xpath expr="//div[hasclass('row')]//div[@name='address_same_as_shipping']//t[@t-set='address']" position="inside">
            <div t-if="o.partner_id.l10n_ca_pst">PST: <span t-field="o.partner_id.l10n_ca_pst"/></div>
        </xpath>
        <xpath expr="//div[hasclass('row')]//div[@name='no_shipping']//t[@t-set='address']" position="inside">
            <div t-if="o.partner_id.l10n_ca_pst">PST: <span t-field="o.partner_id.l10n_ca_pst"/></div>
        </xpath>
    </template>
</odoo>

```

## File: views\report_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="pst_external_layout">
        <li t-if="company.account_fiscal_country_id.code == 'CA' and company.l10n_ca_pst">
            PST: <span t-field="company.l10n_ca_pst"/>
        </li>
    </template>

    <template id="l10n_ca_external_layout_standard" inherit_id="web.external_layout_standard">
        <xpath expr="//ul[@name='company_address_list']" position="inside">
            <t t-call="l10n_ca.pst_external_layout"/>
        </xpath>
    </template>

    <template id="l10n_ca_external_layout_bold" inherit_id="web.external_layout_bold">
        <xpath expr="//ul[@name='company_address_list']" position="inside">
            <t t-call="l10n_ca.pst_external_layout"/>
        </xpath>
    </template>

    <template id="l10n_ca_external_layout_boxed" inherit_id="web.external_layout_boxed">
        <xpath expr="//ul[@name='company_address_list']" position="inside">
            <t t-call="l10n_ca.pst_external_layout"/>
        </xpath>
    </template>

    <template id="l10n_ca_external_layout_striped" inherit_id="web.external_layout_striped">
        <xpath expr="//ul[@name='company_address_list']" position="inside">
            <t t-call="l10n_ca.pst_external_layout"/>
        </xpath>
    </template>

    <template id="l10n_ca_external_layout_bubble" inherit_id="web.external_layout_bubble">
        <xpath expr="//ul[@name='company_address_list']" position="inside">
            <t t-call="l10n_ca.pst_external_layout"/>
        </xpath>
    </template>

    <template id="l10n_ca_external_layout_wave" inherit_id="web.external_layout_wave">
        <xpath expr="//ul[@name='company_address_list']" position="inside">
            <t t-call="l10n_ca.pst_external_layout"/>
        </xpath>
    </template>

    <template id="l10n_ca_external_layout_folder" inherit_id="web.external_layout_folder">
        <xpath expr="//div[hasclass('o_folder_company_info')]" position="after">
            <div t-if="company.account_fiscal_country_id.code == 'CA' and company.l10n_ca_pst">
                PST: <span t-field="company.l10n_ca_pst"/>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: views\res_company_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_company_form_inherit_ca" model="ir.ui.view">
            <field name="name">res.company.form.inherit.l10n.ca</field>
            <field name="model">res.company</field>
            <field name="inherit_id" ref="base.view_company_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='vat']" position="after">
                    <field name="l10n_ca_pst" invisible="country_code != 'CA'"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\res_partner_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_partner_form_inherit_ca" model="ir.ui.view">
            <field name="name">res.partner.form.inherit.l10n.ca</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="account.view_partner_property_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='vat']" position="after">
                    <field name="l10n_ca_pst" invisible="'CA' not in fiscal_country_codes"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

