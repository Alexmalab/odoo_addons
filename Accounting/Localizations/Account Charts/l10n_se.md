# Odoo Module: l10n_se

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Sweden - Accounting',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['se'],
    'version': '1.1',
    'author': 'XCLUDE, Odoo S.A.',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Swedish Accounting
------------------

This is the base module to manage the accounting chart for Sweden in Odoo.
It also includes the invoice OCR payment reference handling.
    """,
    'depends': [
        'account',
        'base_vat',
    ],
    'auto_install': ['account'],
    'data': [
        'data/account.account.tag.csv',
        'data/account_tax_report_data.xml',
        "data/res_country_data.xml",
        'views/partner_view.xml',
        'views/account_journal_view.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.tag.csv

```csv
id,name,applicability
account_tag_0,Subscribed but unpaid capital,accounts
account_tag_1,Intangible non-current assets,accounts
account_tag_2,Tangible non-current assets,accounts
account_tag_3,Financial assets,accounts
account_tag_4,Inventory,accounts
account_tag_5,Receivables,accounts
account_tag_6,Short-term investment,accounts
account_tag_7,Cash and bank accounts,accounts
account_tag_8,Share capital,accounts
account_tag_9,Share premium funds,accounts
account_tag_10,Reevaluation funds,accounts
account_tag_11,Misc. equity funds,accounts
account_tag_12,Balanced gain/loss,accounts
account_tag_13,Equity at the start of the financial year,accounts
account_tag_14,Deposits or withdrawals during the year,accounts
account_tag_15,Changes in the equity funds,accounts
account_tag_16,Changes in the fair value reserves,accounts
account_tag_17,Year-end results,accounts
account_tag_18,Untaxed reserves,accounts
account_tag_19,Provisions,accounts
account_tag_20,Bond loans,accounts
account_tag_21,Liabilities to credit institutions,accounts
account_tag_22,Advances from customers,accounts
account_tag_23,Account payables,accounts
account_tag_24,Liabilities to group companies,accounts
account_tag_25,Exchange liabilities,accounts
account_tag_26,Liabilities to associated companies and jointly controlled companies,accounts
account_tag_27,Liabilities to other companies in which there is an ownership interest,accounts
account_tag_28,Tax liabilities,accounts
account_tag_29,Other liabilities,accounts
account_tag_30,Accrued expenses and prepaid incomes,accounts
account_tag_31,"Operating income, stock changes, etc.",accounts
account_tag_32,Operating costs,accounts
account_tag_33,Financial items,accounts
account_tag_34,Year-end appropriations,accounts
account_tag_35,Taxes on results,accounts
account_tag_36,Results,accounts
account_tag_37,Dedicated funds,accounts
account_tag_38,Reserve funds,accounts
account_tag_39,Other equity,accounts
account_tag_40,Paid-in and issued contributions,accounts

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">skatterapport</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.se"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_title_sales" model="account.report.line">
                <field name="name">Block A – Momspliktig försäljning eller uttag exklusive moms</field>
                <field name="code">se_a</field>
                <field name="aggregation_formula">se_05.balance + se_06.balance + se_07.balance + se_08.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_05" model="account.report.line">
                        <field name="name">Fält 05 – Momspliktig försäljning som inte ingår i fält 06, 07 eller 08</field>
                        <field name="code">se_05</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_05_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_05</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_06" model="account.report.line">
                        <field name="name">Fält 06 – Momspliktiga uttag</field>
                        <field name="code">se_06</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_06_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_06</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_07" model="account.report.line">
                        <field name="name">Fält 07 – Beskattningsunderlag vid vinstmarginalbeskattning</field>
                        <field name="code">se_07</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_07_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_07</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_08" model="account.report.line">
                        <field name="name">Fält 08 – Hyresinkomster vid frivillig skattskyldighet</field>
                        <field name="code">se_08</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_08_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_08</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_output_vat_sales" model="account.report.line">
                <field name="name">Block B – Utgående moms på försäljning eller uttag i fält 05–08</field>
                <field name="code">se_b</field>
                <field name="aggregation_formula">se_10.balance + se_11.balance + se_12.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_10" model="account.report.line">
                        <field name="name">Fält 10 – Utgående moms 25 %</field>
                        <field name="code">se_10</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_11" model="account.report.line">
                        <field name="name">Fält 11 – Utgående moms 12 %</field>
                        <field name="code">se_11</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_11</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_12" model="account.report.line">
                        <field name="name">Fält 12 – Utgående moms 6 %</field>
                        <field name="code">se_12</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_12</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_purchases" model="account.report.line">
                <field name="name">Block C – Momspliktiga inköp vid omvänd skattskyldighet</field>
                <field name="code">se_c</field>
                <field name="aggregation_formula">se_20.balance + se_21.balance + se_22.balance + se_23.balance + se_24.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_20" model="account.report.line">
                        <field name="name">Fält 20 – Inköp av varor från annat EU-land</field>
                        <field name="code">se_20</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_20_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_20</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_21" model="account.report.line">
                        <field name="name">Fält 21 – Inköp av tjänster från ett annat EU-land, enligt huvudregeln</field>
                        <field name="code">se_21</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_21_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_21</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_22" model="account.report.line">
                        <field name="name">Fält 22 – Inköp av tjänster från länder utanför EU</field>
                        <field name="code">se_22</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_22_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_22</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_23" model="account.report.line">
                        <field name="name">Fält 23 – Inköp av varor i Sverige</field>
                        <field name="code">se_23</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_23_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_23</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_24" model="account.report.line">
                        <field name="name">Fält 24 – Övriga inköp av tjänster</field>
                        <field name="code">se_24</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_24_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_24</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_output_vat_purchases" model="account.report.line">
                <field name="name">Block D – Utgående moms på inköp i fält 20–24</field>
                <field name="code">se_d</field>
                <field name="aggregation_formula">se_30.balance + se_31.balance + se_32.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_30" model="account.report.line">
                        <field name="name">Fält 30 – Utgående moms 25 %</field>
                        <field name="code">se_30</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_30_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_30</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_31" model="account.report.line">
                        <field name="name">Fält 31 – Utgående moms 12 %</field>
                        <field name="code">se_31</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_31_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_31</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_32" model="account.report.line">
                        <field name="name">Fält 32 – Utgående moms 6 %</field>
                        <field name="code">se_32</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_32_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_32</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_imports" model="account.report.line">
                <field name="name">Block H - moms vid import</field>
                <field name="code">se_h</field>
                <field name="aggregation_formula">se_50.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_50" model="account.report.line">
                        <field name="name">Fält 50 - Beskattningsunderlag vid import</field>
                        <field name="code">se_50</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_50_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_50</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_output_vat_imports" model="account.report.line">
                <field name="name">Block I - Utgående moms på import i fält 50</field>
                <field name="code">se_i</field>
                <field name="aggregation_formula">se_60.balance + se_61.balance + se_62.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_60" model="account.report.line">
                        <field name="name">Fält 60 – Utgående moms 25 %</field>
                        <field name="code">se_60</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_60_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_60</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_61" model="account.report.line">
                        <field name="name">Fält 61 – Utgående moms 12 %</field>
                        <field name="code">se_61</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_61_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_61</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_62" model="account.report.line">
                        <field name="name">Fält 62 – Utgående moms 6 %</field>
                        <field name="code">se_62</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_62_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_62</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_exempt_sales" model="account.report.line">
                <field name="name">Block E – Försäljning m.m. som är undantagen från moms</field>
                <field name="code">se_e</field>
                <field name="aggregation_formula">se_35.balance + se_36.balance + se_37.balance + se_38.balance + se_39.balance + se_40.balance + se_41.balance + se_42.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_35" model="account.report.line">
                        <field name="name">Fält 35 – Försäljning av varor till ett annat EU-land</field>
                        <field name="code">se_35</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_35_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_35</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_36" model="account.report.line">
                        <field name="name">Fält 36 – Försäljning av varor utanför EU</field>
                        <field name="code">se_36</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_36_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_36</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_37" model="account.report.line">
                        <field name="name">Fält 37 – Mellanmans inköp av varor vid trepartshandel</field>
                        <field name="code">se_37</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_37_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_37</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_38" model="account.report.line">
                        <field name="name">Fält 38 – Mellanmans försäljning av varor vid trepartshandel</field>
                        <field name="code">se_38</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_38_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_38</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_39" model="account.report.line">
                        <field name="name">Fält 39 – Försäljning av tjänster till en beskattningsbar person (näringsidkare) i ett annat EU-land, enligt huvudregeln </field>
                        <field name="code">se_39</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_39_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_39</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_40" model="account.report.line">
                        <field name="name">Fält 40 – Övrig försäljning av tjänster omsatta utanför Sverige</field>
                        <field name="code">se_40</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_40_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_40</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_41" model="account.report.line">
                        <field name="name">Fält 41 – Försäljning när köparen är skattskyldig i Sverige</field>
                        <field name="code">se_41</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_41_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_41</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_42" model="account.report.line">
                        <field name="name">Fält 42 – Övrig försäljning m.m.</field>
                        <field name="code">se_42</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_42_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_42</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_input_vat" model="account.report.line">
                <field name="name">Block F – Ingående moms</field>
                <field name="code">se_f</field>
                <field name="aggregation_formula">se_48.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_48" model="account.report.line">
                        <field name="name">Fält 48 – Ingående moms att dra av</field>
                        <field name="code">se_48</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_48_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_48</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_vat_debt_credit" model="account.report.line">
                <field name="name">Block G – Moms att betala eller få tillbaka</field>
                <field name="code">se_g</field>
                <field name="aggregation_formula">se_b.balance+se_i.balance+se_d.balance-se_f.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_49" model="account.report.line">
                        <field name="name">Fält 49 – Moms att betala eller få tillbaka</field>
                        <field name="code">se_49</field>
                        <field name="aggregation_formula">se_b.balance+se_i.balance+se_d.balance-se_f.balance</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\res_country_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="se_partner_address_form" model="ir.ui.view">
        <field name="name">se.partner.form.address</field>
        <field name="model">res.partner</field>
        <field name="priority" eval="900"/>
        <field name="arch" type="xml">
            <form>
                <div class="o_address_format">
                    <field name="parent_id" invisible="1"/>
                    <field name="type" invisible="1"/>
                    <field name="street" placeholder="Street" class="o_address_street"
                            readonly="type == 'contact' and parent_id"/>
                    <field name="street2" placeholder="Neighborhood" class="o_address_street"
                            readonly="type == 'contact' and parent_id"/>
                    <field name="zip" placeholder="ZIP" class="o_address_zip"
                            readonly="type == 'contact' and parent_id"/>
                    <field name="city" placeholder="City" class="o_address_city"
                            readonly="type == 'contact' and parent_id"/>
                    <field name="state_id" class="o_address_state" placeholder="State..." options='{"no_open": True}'
                            readonly="type == 'contact' and parent_id"/>
                    <field name="country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}'
                            readonly="type == 'contact' and parent_id"/>
                    <field name="state_id" class="o_address_state" placeholder="State..." options='{"no_open": True}'
                            readonly="type == 'contact' and parent_id"/>
                </div>
            </form>
        </field>
    </record>
    <record id="base.se" model="res.country">
        <field name="address_view_id" ref="se_partner_address_form" />
        <field name="address_format" eval="'%(street)s\n%(street2)s\n%(zip)s %(city)s %(state_code)s\n%(country_name)s'"/>
    </record>
</odoo>

```

## File: data\template\account.account-se.csv

```csv
"id","code","name","account_type","tag_ids","reconcile","name@sv_SE"
"a1030","1030","Patents","asset_non_current","l10n_se.account_tag_1","False","Patent"
"a1039","1039","Accumulated amortisation of patents","asset_non_current","l10n_se.account_tag_1","False","Ackumulerade avskrivningar på patent"
"a1060","1060","Tenements, leaseholds and similar rights","asset_non_current","l10n_se.account_tag_1","False","Hyresrätter, tomträtter och liknande"
"a1069","1069","Accumulated amortisation of tenancies, leasehold and similar rights","asset_non_current","l10n_se.account_tag_1","False","Ackumulerade avskrivningar på hyresrätter, tomträtter och liknande"
"a1110","1110","Buildings","asset_fixed","l10n_se.account_tag_2","False","Byggnader"
"a1119","1119","Accumulated depreciation of buildings","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade avskrivningar på byggnader"
"a1130","1130","Land","asset_fixed","l10n_se.account_tag_2","False","Mark"
"a1150","1150","Land improvements","asset_fixed","l10n_se.account_tag_2","False","Markanläggningar"
"a1159","1159","Accumulated depreciation of land improvements","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade avskrivningar på markanläggningar"
"a1210","1210","Machinery and other technical equipment","asset_fixed","l10n_se.account_tag_2","False","Maskiner och andra tekniska anläggningar"
"a1219","1219","Accumulated depreciation of machinery and equipment","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade avskrivningar på maskiner och andra tekniska anläggningar"
"a1220","1220","Equipment and tools","asset_fixed","l10n_se.account_tag_2","False","Inventarier och verktyg"
"a1229","1229","Accumulated depreciation of equipment and tools","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade avskrivningar på inventarier och verktyg"
"a1240","1240","Vehicles and other transport equipment","asset_fixed","l10n_se.account_tag_2","False","Bilar och andra transportmedel"
"a1249","1249","Accumulated depreciation of cars and other transport equipment","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade avskrivningar på bilar och andra transportmedel"
"a1250","1250","Computers","asset_fixed","l10n_se.account_tag_2","False","Datorer"
"a1259","1259","Accumulated depreciation of computers","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade avskrivningar på datorer"
"a1290","1290","Other tangible fixed assets","asset_fixed","l10n_se.account_tag_2","False","Övriga materiella anläggningstillgångar"
"a1291","1291","Art and similar assets","asset_fixed","l10n_se.account_tag_2","False","Konst och liknande tillgångar"
"a1299","1299","Accumulated depreciation of other tangible fixed assets","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade avskrivningar på övriga materiella anläggningstillgångar"
"a1350","1350","Shares and securities in other enterprises","asset_non_current","l10n_se.account_tag_3","False","Andelar och värdepapper i andra företag"
"a1380","1380","Other long-term receivables","asset_non_current","l10n_se.account_tag_3","False","Andra långfristiga fordringar"
"a1410","1410","Stocks of raw materials","asset_current","l10n_se.account_tag_4","False","Lager av råvaror"
"a1419","1419","Change in stocks of raw materials","asset_current","l10n_se.account_tag_4","False","Förändring av lager av råvaror"
"a1440","1440","Work in progress","asset_current","l10n_se.account_tag_4","False","Produkter i arbete"
"a1449","1449","Change in work in progress","asset_current","l10n_se.account_tag_4","False","Förändring av produkter i arbete"
"a1450","1450","Stocks of finished goods","asset_current","l10n_se.account_tag_4","False","Lager av färdiga varor"
"a1459","1459","Change in stocks of finished goods","asset_current","l10n_se.account_tag_4","False","Förändring av lager av färdiga varor"
"a1460","1460","Stocks of goods for resale","asset_current","l10n_se.account_tag_4","False","Lager av handelsvaror"
"a1469","1469","Change in stocks of goods for resale","asset_current","l10n_se.account_tag_4","False","Förändring av lager av handelsvaror"
"a1470","1470","Work in progress","asset_current","l10n_se.account_tag_4","False","Pågående arbeten"
"a1479","1479","Change in work in progress","asset_current","l10n_se.account_tag_4","False","Förändring av Pågående arbete"
"a1480","1480","Advances for goods and services","asset_prepayments","l10n_se.account_tag_4","False","Förskott för varor och tjänster"
"a1490","1490","Other inventory assets","asset_prepayments","l10n_se.account_tag_4","False","Övriga lagertillgångar"
"a1510","1510","Trade receivables","asset_receivable","l10n_se.account_tag_5","True","Kundfordringar"
"a1513","1513","Trade receivables - split invoice","asset_receivable","l10n_se.account_tag_5","True","Kundfordringar - delad faktura"
"a1519","1519","Impairment of trade receivables","asset_receivable","l10n_se.account_tag_5","True","Nedskrivning av kundfordringar"
"a1580","1580","Receivables for credit cards and vouchers","asset_receivable","l10n_se.account_tag_5","True","Fordringar för kontokort och kuponger"
"a1610","1610","Short-term receivables from employees","asset_receivable","l10n_se.account_tag_5","True","Kortfristiga fordringar hos anställda"
"a1630","1630","Offset for taxes and duties (tax account)","asset_current","l10n_se.account_tag_5","False","Avräkning för skatter och avgifter (skattekonto)"
"a1640","1640","Tax receivables","asset_current","l10n_se.account_tag_5","False","Skattefordringar"
"a1650","1650","VAT receivable","asset_current","l10n_se.account_tag_5","False","Momsfordran"
"a1680","1680","Other current receivables","asset_current","l10n_se.account_tag_5","False","Andra kortfristiga fordringar"
"a1710","1710","Prepaid rentals","asset_prepayments","l10n_se.account_tag_5","False","Förutbetalda hyreskostnader"
"a1720","1720","Prepaid leasing fees, current portion","asset_prepayments","l10n_se.account_tag_5","False","Förutbetalda leasingavgifter, kortfristig del"
"a1730","1730","Prepaid insurance premiums","asset_prepayments","l10n_se.account_tag_5","False","Förutbetalda försäkringspremier"
"a1740","1740","Prepaid interest expenses","asset_prepayments","l10n_se.account_tag_5","False","Förutbetalda räntekostnader"
"a1750","1750","Accrued rental income","asset_current","l10n_se.account_tag_5","False","Upplupna hyresintäkter"
"a1760","1760","Accrued interest income","asset_current","l10n_se.account_tag_5","False","Upplupna ränteintäkter"
"a1790","1790","Other prepaid expenses and accrued income","asset_current","l10n_se.account_tag_5","False","Övriga förutbetalda kostnader och upplupna intäkter"
"a1810","1810","Shares in listed companies","asset_current","l10n_se.account_tag_6","False","Andel i börsnoterade företag"
"a1880","1880","Other short-term investments","asset_current","l10n_se.account_tag_6","False","Andra kortfristiga placeringar"
"a1890","1890","Impairment of short-term investments","asset_current","l10n_se.account_tag_6","False","Nedskrivning av kortfristiga placeringar"
"a1910","1910","Cash in hand","asset_cash","l10n_se.account_tag_7","False","Kassa"
"a1920","1920","PlusGiro","asset_cash","l10n_se.account_tag_7","False","PlusGiro"
"a1930","1930","Company account/checking account/business account","asset_cash","l10n_se.account_tag_7","False","Företagskonto/checkkonto/affärskonto"
"a1940","1940","Other bank accounts","asset_cash","l10n_se.account_tag_7","False","Övriga bankkonton"
"a2010","2010","Equity capital, partner 1","equity","l10n_se.account_tag_13","False","Eget kapital, delägare 1"
"a2011","2011","Own withdrawals of goods","equity","l10n_se.account_tag_14","True","Egna varuuttag"
"a2013","2013","Other own withdrawals","equity","l10n_se.account_tag_14","True","Övriga egna uttag"
"a2017","2017","Capital contribution for the year","equity","l10n_se.account_tag_14","True","Årets kapitaltillskott"
"a2018","2018","Other own deposits","equity","l10n_se.account_tag_14","True","Övriga egna insättningar"
"a2019","2019","Profit for the year, partners","equity","l10n_se.account_tag_17","False","Årets resultat, delägare"
"a2020","2020","Equity, shareholder 2","equity","l10n_se.account_tag_13","False","Eget kapital, delägare 2"
"a2030","2030","Equity, shareholder 3","equity","l10n_se.account_tag_13","False","Eget kapital, delägare 3"
"a2040","2040","Equity, shareholder 4","equity","l10n_se.account_tag_13","False","Eget kapital, delägare 4"
"a2060","2060","Equity in non-profit organisations, foundations and registered religious communities","equity","l10n_se.account_tag_13","False","Eget kapital i ideella föreningar, stiftelser och registrerade trossamfund"
"a2070","2070","Restricted funds","equity","l10n_se.account_tag_37","False","Ändamålsbestämda medel"
"a2081","2081","Share capital","equity","l10n_se.account_tag_8","False","Aktiekapital"
"a2083","2083","Members' contributions","equity","l10n_se.account_tag_40","False","Medlemsinsatser"
"a2086","2086","Reserve fund","equity","l10n_se.account_tag_38","False","Reservfond"
"a2090","2090","Unrestricted equity","equity","l10n_se.account_tag_39","False","Fritt eget kapital"
"a2091","2091","Profit or loss brought forward","equity","l10n_se.account_tag_12","False","Balanserad vinst eller förlust"
"a2098","2098","Profit or loss from previous year","equity","l10n_se.account_tag_11","False","Vinst eller förlust från föregående år"
"a2099","2099","Profit or loss for the year","equity","l10n_se.account_tag_17","False","Årets resultat"
"a2120","2120","Accrual fund 2020","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2020"
"a2121","2121","Accrual fund 2021","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2021"
"a2122","2122","Accrual fund 2022","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2022"
"a2123","2123","Accrual fund 2023","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2023"
"a2124","2124","Accrual fund 2024","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2024"
"a2126","2126","Accrual fund 2016","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2016"
"a2127","2127","Accrual fund 2017","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2017"
"a2128","2128","Accrual fund 2018","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2018"
"a2129","2129","Accrual fund 2019","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2019"
"a2150","2150","Accumulated excess depreciation","liability_non_current","l10n_se.account_tag_18","False","Ackumulerade överavskrivningar"
"a2210","2210","Provisions for pensions under the Social Security Act","liability_non_current","l10n_se.account_tag_19","False","Avsättningar för pensioner enligt tryggandelagen"
"a2220","2220","Provisions for guarantees","liability_non_current","l10n_se.account_tag_19","False","Avsättningar för garantier"
"a2290","2290","Other provisions","liability_non_current","l10n_se.account_tag_19","False","Övriga avsättningar"
"a2330","2330","Overdraft facility","liability_non_current","l10n_se.account_tag_21","False","Checkräkningskredit"
"a2350","2350","Other long-term liabilities to credit institutions","liability_non_current","l10n_se.account_tag_21","False","Andra långfristiga skulder till kreditinstitut"
"a2390","2390","Other long-term liabilities","liability_non_current","l10n_se.account_tag_29","False","Övriga långfristiga skulder"
"a2393","2393","Loans from related parties, long-term part","liability_non_current","l10n_se.account_tag_29","False","Lån från närstående personer, långfristiga del"
"a2410","2410","Other short-term loan liabilities to credit institutions","liability_current","l10n_se.account_tag_21","False","Andra kortfristiga låneskulder till kreditinstitut"
"a2420","2420","Advances from customers","liability_current","l10n_se.account_tag_22","False","Förskott från kunder"
"a2440","2440","Trade payables","liability_payable","l10n_se.account_tag_23","True","Leverantörsskulder"
"a2480","2480","Bank overdraft, short-term","liability_payable","l10n_se.account_tag_21","True","Checkräkningskredit, kortfristig"
"a2490","2490","Other current liabilities to credit institutions, customers and suppliers","liability_current","l10n_se.account_tag_21","False","Övriga kortfristiga skulder till kreditinstitut, kunder och leverantörer"
"a2510","2510","Tax liabilities","liability_current","l10n_se.account_tag_28","False","Skatteskulder"
"a2610","2610","Outgoing VAT, 25 %","liability_current","l10n_se.account_tag_29","False","Utgående moms, 25 %"
"a2611","2611","Outgoing VAT on sales within Sweden, 25%","liability_current","l10n_se.account_tag_29","False","Utgående moms på försäljning inom Sverige, 25 %"
"a2612","2612","Outgoing VAT on own withdrawals, 25%","liability_current","l10n_se.account_tag_29","False","Utgående moms på egna uttag, 25 %"
"a2613","2613","Output VAT for rentals, 25 %","liability_current","l10n_se.account_tag_29","False","Utgående moms för uthyrning, 25 %"
"a2614","2614","Output VAT on reverse charge, 25 per cent","liability_current","l10n_se.account_tag_29","False","Utgående moms omvänd skattskyldighet, 25 %"
"a2615","2615","Outgoing VAT on import of goods, 25 %","liability_current","l10n_se.account_tag_29","False","Utgående moms import av varor, 25 %"
"a2616","2616","Output VAT VMB 25 per cent","liability_current","l10n_se.account_tag_29","False","Utgående moms VMB 25 %"
"a2620","2620","Output VAT, 12 %","liability_current","l10n_se.account_tag_29","False","Utgående moms, 12 %"
"a2621","2621","Output VAT on sales within Sweden, 12 %","liability_current","l10n_se.account_tag_29","False","Utgående moms på försäljning inom Sverige, 12 %"
"a2622","2622","Output VAT on own withdrawals, 12 %","liability_current","l10n_se.account_tag_29","False","Utgående moms på egna uttag, 12 %"
"a2623","2623","Output VAT for rentals, 12 %","liability_current","l10n_se.account_tag_29","False","Utgående moms för uthyrning, 12 %"
"a2624","2624","Output VAT reverse charge, 12 %","liability_current","l10n_se.account_tag_29","False","Utgående moms omvänd skattskyldighet, 12 %"
"a2625","2625","Output VAT on import of goods, 12 %","liability_current","l10n_se.account_tag_29","False","Utgående moms import av varor, 12 %"
"a2626","2626","Output VAT on VAT, 12 %","liability_current","l10n_se.account_tag_29","False","Utgående moms VMB, 12 %"
"a2630","2630","Output VAT, 6 %","liability_current","l10n_se.account_tag_29","False","Utgående moms, 6 %"
"a2631","2631","Output VAT on sales within Sweden, 6 %","liability_current","l10n_se.account_tag_29","False","Utgående moms på försäljning inom Sverige, 6 %"
"a2632","2632","Output VAT on own withdrawals, 6 %","liability_current","l10n_se.account_tag_29","False","Utgående moms på egna uttag, 6 %"
"a2633","2633","Output VAT for rentals, 6 per cent","liability_current","l10n_se.account_tag_29","False","Utgående moms för uthyrning, 6 %"
"a2634","2634","Output VAT on reverse charge, 6 %","liability_current","l10n_se.account_tag_29","False","Utgående moms omvänd skattskyldighet, 6 %"
"a2635","2635","Output VAT on imports of goods, 6%","liability_current","l10n_se.account_tag_29","False","Utgående moms import av varor, 6 %"
"a2636","2636","Output VAT VMB 6 per cent","liability_current","l10n_se.account_tag_29","False","Utgående moms VMB 6 %"
"a2640","2640","Input VAT","liability_current","l10n_se.account_tag_29","False","Ingående moms"
"a2641","2641","Debited input VAT","liability_current","l10n_se.account_tag_29","False","Debiterad ingående moms"
"a2642","2642","Debited input VAT in connection with voluntary tax liability","liability_current","l10n_se.account_tag_29","False","Debiterad ingående moms i anslutning till frivillig skattskyldighet"
"a2645","2645","Calculated input VAT on acquisitions from abroad","liability_current","l10n_se.account_tag_29","False","Beräknad ingående moms på förvärv från utlandet"
"a2646","2646","Input VAT on rentals","liability_current","l10n_se.account_tag_29","False","Ingående moms på uthyrning"
"a2647","2647","Input VAT on reverse charge goods and services in Sweden.","liability_current","l10n_se.account_tag_29","False","Ingående moms omvänd skattskyldighet varor och tjänster i Sverige"
"a2648","2648","Dormant input VAT","liability_current","l10n_se.account_tag_29","False","Vilande ingående moms"
"a2649","2649","Input VAT, mixed activities","liability_current","l10n_se.account_tag_29","False","Ingående moms, blandad verksamhet"
"a2650","2650","Accounting account for VAT","liability_current","l10n_se.account_tag_29","False","Redovisningskonto för moms"
"a2710","2710","Personnel tax","liability_current","l10n_se.account_tag_29","False","Personalskatt"
"a2730","2730","Statutory social security contributions and special payroll tax","liability_current","l10n_se.account_tag_29","False","Lagstadgade sociala avgifter och särskild löneskatt"
"a2740","2740","Agreed social security contributions","liability_current","l10n_se.account_tag_29","False","Avtalade sociala avgifter"
"a2790","2790","Other payroll deductions","liability_current","l10n_se.account_tag_29","False","Övriga löneavdrag"
"a2820","2820","Current liabilities to employees","liability_payable","l10n_se.account_tag_29","True","Kortfristiga skulder till anställda"
"a2840","2840","Short-term loan liabilities","liability_payable","l10n_se.account_tag_29","True","Kortfristiga låneskulder"
"a2890","2890","Other current liabilities","liability_current","l10n_se.account_tag_29","False","Övriga kortfristiga skulder"
"a2910","2910","Accrued wages and salaries","liability_current","l10n_se.account_tag_30","False","Upplupna löner"
"a2920","2920","Accrued holiday pay","liability_current","l10n_se.account_tag_30","False","Upplupna semesterlöner"
"a2940","2940","Accrued statutory social security and other contributions","liability_current","l10n_se.account_tag_30","False","Upplupna lagstadgade sociala och andra avgifter"
"a2950","2950","Accrued contractual social security contributions","liability_current","l10n_se.account_tag_30","False","Upplupna avtalade sociala avgifter"
"a2960","2960","Accrued interest expenses","liability_current","l10n_se.account_tag_30","False","Upplupna räntekostnader"
"a2970","2970","Deferred income","liability_current","l10n_se.account_tag_30","False","Förutbetalda intäkter"
"a2990","2990","Other accrued expenses and deferred income","liability_current","l10n_se.account_tag_30","False","Övriga upplupna kostnader och förutbetalda intäkter"
"a2999","2999","OBS account","liability_current","l10n_se.account_tag_30","False","OBS-konto"
"a3000","3000","Sales within Sweden","income","l10n_se.account_tag_31","False","Försäljning inom Sverige"
"a3001","3001","Sales within Sweden, 25% VAT","income","l10n_se.account_tag_31","False","Försäljning inom Sverige, 25 % moms"
"a3002","3002","Sales within Sweden, 12% VAT","income","l10n_se.account_tag_31","False","Försäljning inom Sverige, 12 % moms"
"a3003","3003","Sales within Sweden, 6% VAT","income","l10n_se.account_tag_31","False","Försäljning inom Sverige, 6 % moms"
"a3004","3004","Sales within Sweden, VAT free","income","l10n_se.account_tag_31","False","Försäljning inom Sverige, momsfri"
"a3100","3100","Sale of goods outside the EU","income","l10n_se.account_tag_31","False","Försäljning av varor utanför EU"
"a3105","3105","Sale of goods to a non-EU country","income","l10n_se.account_tag_31","False","Försäljning varor till land utanför EU"
"a3106","3106","Sale of goods to another EU country, subject to VAT","income","l10n_se.account_tag_31","False","Försäljning varor till annat EU-land, momspliktig"
"a3108","3108","Sale of goods to another EU country, exempt from VAT","income","l10n_se.account_tag_31","False","Försäljning varor till annat EU-land, momsfri"
"a3200","3200","Sales of VAT and reverse charge","income","l10n_se.account_tag_31","False","Försäljning VMB och omvänd moms"
"a3211","3211","Sale of positive VAT 25 per cent","income","l10n_se.account_tag_31","False","Försäljning positiv VMB 25 %"
"a3212","3212","Sales negative VAT 25 per cent","income","l10n_se.account_tag_31","False","Försäljning negativ VMB 25 %"
"a3231","3231","Sales in the construction sector, reverse charge VAT","income","l10n_se.account_tag_31","False","Försäljning inom byggsektorn, omvänd skatteskyldighet moms"
"a3300","3300","Sales of services outside Sweden","income","l10n_se.account_tag_31","False","Försäljning av tjänster utanför Sverige"
"a3305","3305","Sale of services to a country outside the EU","income","l10n_se.account_tag_31","False","Försäljning av tjänster till land utanför EU"
"a3308","3308","Sale of services to another EU country","income","l10n_se.account_tag_31","False","Försäljning av tjänster till annat EU-land"
"a3400","3400","Sales, own withdrawals","income","l10n_se.account_tag_31","False","Försäljning, egna uttag"
"a3401","3401","Own withdrawals subject to VAT, 25 %","income","l10n_se.account_tag_31","False","Egna uttag momspliktiga, 25 %"
"a3402","3402","Own withdrawals subject to VAT, 12 %","income","l10n_se.account_tag_31","False","Egna uttag momspliktiga, 12 %"
"a3403","3403","Own withdrawals subject to VAT, 6%","income","l10n_se.account_tag_31","False","Egna uttag momspliktiga, 6 %"
"a3404","3404","Own withdrawals, VAT free","income","l10n_se.account_tag_31","False","Egna uttag, momsfria"
"a3500","3500","Invoiced costs (group account)","income","l10n_se.account_tag_31","False","Fakturerade kostnader (gruppkonto)"
"a3510","3510","Invoiced packaging","income","l10n_se.account_tag_31","False","Fakturerat emballage"
"a3520","3520","Invoiced freight","income","l10n_se.account_tag_31","False","Fakturerade frakter"
"a3521","3521","Invoiced freight, EU country","income","l10n_se.account_tag_31","False","Fakturerade frakter, EU-land"
"a3522","3522","Invoiced freight, export","income","l10n_se.account_tag_31","False","Fakturerade frakter, export"
"a3530","3530","Invoiced customs and forwarding costs, etc.","income","l10n_se.account_tag_31","False","Fakturerad tull- och speditionskostnader m.m."
"a3540","3540","Invoiced charges","income","l10n_se.account_tag_31","False","Faktureringsavgifter"
"a3541","3541","Billing charges, EU country","income","l10n_se.account_tag_31","False","Faktureringsavgifter, EU-land"
"a3542","3542","Billing charges, export","income","l10n_se.account_tag_31","False","Faktureringsavgifter, export"
"a3600","3600","Ancillary operating income (group account)","income","l10n_se.account_tag_31","False","Rörelsens sidointäkter (gruppkonto)"
"a3730","3730","Discounts granted","income","l10n_se.account_tag_31","False","Lämnade rabatter"
"a3740","3740","Island and crown equalisation","income","l10n_se.account_tag_31","False","Öres- och kronutjämning"
"a3800","3800","Capitalised work for own account (group account)","income","l10n_se.account_tag_31","False","Aktiverat arbete för egen räkning (gruppkonto)"
"a3900","3900","Other operating income (group account)","income_other","l10n_se.account_tag_31","False","Övriga rörelseintäkter (gruppkonto)"
"a3913","3913","Rental income voluntarily subject to VAT","income_other","l10n_se.account_tag_31","False","Frivilligt momspliktiga hyresintäkter"
"a3960","3960","Foreign exchange gains on receivables and liabilities of an operating nature","income_other","l10n_se.account_tag_31","False","Valutakursvinster på fordringar och skulder av rörelsekaraktär"
"a3970","3970","Gains on the disposal of intangible and tangible fixed assets","income_other","l10n_se.account_tag_31","False","Vinst vid avyttring av immateriella och materiella anläggningstillgångar"
"a3980","3980","Government grants received, etc.","income_other","l10n_se.account_tag_31","False","Erhållna offentliga stöd m.m."
"a4000","4000","Purchases of goods from Sweden","expense_direct_cost","l10n_se.account_tag_32","False","Inköp av varor från Sverige"
"a4200","4200","Sales of goods sold VMB","expense_direct_cost","l10n_se.account_tag_32","False","Sålda varor VMB"
"a4211","4211","Goods sold positive VAT 25%","expense_direct_cost","l10n_se.account_tag_32","False","Sålda varor positiv VMB 25 %"
"a4212","4212","Goods sold negative VAT 25%","expense_direct_cost","l10n_se.account_tag_32","False","Sålda varor negativ VMB 25 %"
"a4400","4400","Purchases subject to VAT in Sweden","expense_direct_cost","l10n_se.account_tag_32","False","Momspliktiga inköp i Sverige"
"a4415","4415","Goods purchased in Sweden, reverse charge, 25 %","expense_direct_cost","l10n_se.account_tag_32","False","Inköpta varor i Sverige, omvänd skattskyldighet, 25 %"
"a4426","4426","Purchases of services in Sweden, reverse charge, 12 %","expense_direct_cost","l10n_se.account_tag_32","False","Inköp tjänster i Sverige, omvänd skattskyldighet, 12 %"
"a4427","4427","Purchases of services in Sweden, reverse charge, 6%","expense_direct_cost","l10n_se.account_tag_32","False","Inköp tjänster i Sverige, omvänd skattskyldighet, 6 %"
"a4500","4500","Other purchases subject to VAT","expense_direct_cost","l10n_se.account_tag_32","False","Övriga momspliktiga inköp"
"a4515","4515","Purchases of goods from another EU country, 25%","expense_direct_cost","l10n_se.account_tag_32","False","Inköp av varor från annat EU-land, 25 %"
"a4516","4516","Purchases of goods from another EU country, 12 %","expense_direct_cost","l10n_se.account_tag_32","False","Inköp av varor från annat EU-land, 12 %"
"a4517","4517","Purchases of goods from another EU country, 6%","expense_direct_cost","l10n_se.account_tag_32","False","Inköp av varor från annat EU-land, 6 %"
"a4518","4518","Purchase of goods from another EU country, VAT free","expense_direct_cost","l10n_se.account_tag_32","False","Inköp av varor från annat EU-land, momsfri"
"a4531","4531","Purchase of services from a non-EU country, 25%","expense_direct_cost","l10n_se.account_tag_32","False","Inköp av tjänster från ett land utanför EU, 25 %"
"a4532","4532","Purchase of services from outside the EU, 12%","expense_direct_cost","l10n_se.account_tag_32","False","Inköp av tjänster från ett land utanför EU, 12 %"
"a4533","4533","Purchase of services from outside the EU, 6%","expense_direct_cost","l10n_se.account_tag_32","False","Inköp av tjänster från ett land utanför EU, 6 %"
"a4535","4535","Purchase of services from another EU country, 25%","expense_direct_cost","l10n_se.account_tag_32","False","Inköp av tjänster från annat EU-land, 25 %"
"a4536","4536","Purchase of services from another EU country, 12%","expense_direct_cost","l10n_se.account_tag_32","False","Inköp av tjänster från annat EU-land, 12 %"
"a4537","4537","Purchase of services from another EU country, 6%","expense_direct_cost","l10n_se.account_tag_32","False","Inköp av tjänster från annat EU-land, 6 %"
"a4538","4538","Purchase of services from another EU country, VAT free","expense_direct_cost","l10n_se.account_tag_32","False","Inköp av tjänster från annat EU-land, momsfri"
"a4545","4545","Import of goods, 25% VAT","expense_direct_cost","l10n_se.account_tag_32","False","Import av varor, 25 % moms"
"a4546","4546","Import of goods, 12% VAT","expense_direct_cost","l10n_se.account_tag_32","False","Import av varor, 12 % moms"
"a4547","4547","Import of goods, 6% VAT","expense_direct_cost","l10n_se.account_tag_32","False","Import av varor, 6 % moms"
"a4600","4600","Subcontracting and subcontracting (group account)","expense_direct_cost","l10n_se.account_tag_32","False","Legoarbeten och underentreprenader (gruppkonto)"
"a4700","4700","Reduction of purchase prices (group account)","expense_direct_cost","l10n_se.account_tag_32","False","Reduktion av inköpspriser (gruppkonto)"
"a4900","4900","Change in stocks (group account)","expense_direct_cost","l10n_se.account_tag_31","False","Förändring av lager (gruppkonto)"
"a4910","4910","Change in stocks of raw materials","expense_direct_cost","l10n_se.account_tag_32","False","Förändring av lager av råvaror"
"a4920","4920","Change in stocks of additives and consumables","expense_direct_cost","l10n_se.account_tag_32","False","Förändring av lager av tillsatsmaterial och förnödenheter"
"a4940","4940","Change in stocks of work in progress","expense_direct_cost","l10n_se.account_tag_31","False","Förändring produkter i arbete"
"a4950","4950","Change in stocks of finished goods","expense_direct_cost","l10n_se.account_tag_31","False","Förändring av lager av färdiga varor"
"a4960","4960","Change in stocks of goods for resale","expense_direct_cost","l10n_se.account_tag_31","False","Förändring av lager av handelsvaror"
"a4970","4970","Change in work in progress, costs incurred","expense_direct_cost","l10n_se.account_tag_31","False","Förändring pågående arbete, nedlagda kostnader"
"a5010","5010","Rent of premises","expense","l10n_se.account_tag_32","False","Lokalhyra"
"a5020","5020","Electricity for lighting","expense","l10n_se.account_tag_32","False","El för belysning"
"a5030","5030","Heating","expense","l10n_se.account_tag_32","False","Värme"
"a5040","5040","Water and sewerage","expense","l10n_se.account_tag_32","False","Vatten och avlopp"
"a5060","5060","Cleaning and sanitation","expense","l10n_se.account_tag_32","False","Städning och renhållning"
"a5070","5070","Repair and maintenance of premises","expense","l10n_se.account_tag_32","False","Reparation och underhåll av lokaler"
"a5120","5120","Electricity for lighting","expense","l10n_se.account_tag_32","False","El för belysning"
"a5130","5130","Heating","expense","l10n_se.account_tag_32","False","Värme"
"a5140","5140","Water and sewerage","expense","l10n_se.account_tag_32","False","Vatten och avlopp"
"a5160","5160","Cleaning and sanitation","expense","l10n_se.account_tag_32","False","Städning och renhållning"
"a5170","5170","Repair and maintenance of property","expense","l10n_se.account_tag_32","False","Reparation och underhåll av fastighet"
"a5200","5200","Rent of fixed assets (group account)","expense","l10n_se.account_tag_32","False","Hyra av anläggningstillgångar (gruppkonto)"
"a5300","5300","Energy costs (group account)","expense","l10n_se.account_tag_32","False","Energikostnader (gruppkonto)"
"a5410","5410","Consumable inventory","expense","l10n_se.account_tag_32","False","Förbrukningsinventarier"
"a5420","5420","Software","expense","l10n_se.account_tag_32","False","Programvaror"
"a5460","5460","Consumables","expense","l10n_se.account_tag_32","False","Förbrukningsmaterial"
"a5500","5500","Repair and maintenance (group account)","expense","l10n_se.account_tag_32","False","Reparation och underhåll (gruppkonto)"
"a5600","5600","Cost of means of transport (group account)","expense","l10n_se.account_tag_32","False","Kostnader för transportmedel (gruppkonto)"
"a5611","5611","Fuel for passenger cars","expense","l10n_se.account_tag_32","False","Drivmedel för personbilar"
"a5612","5612","Insurance and taxes for passenger cars","expense","l10n_se.account_tag_32","False","Försäkring och skatt för personbilar"
"a5613","5613","Repair and maintenance of passenger cars","expense","l10n_se.account_tag_32","False","Reparation och underhåll av personbilar"
"a5615","5615","Leasing of passenger cars","expense","l10n_se.account_tag_32","False","Leasing av personbilar"
"a5700","5700","Freight and transport (group account)","expense","l10n_se.account_tag_32","False","Frakter och transporter (gruppkonto)"
"a5800","5800","Travelling expenses (group account)","expense","l10n_se.account_tag_32","False","Resekostnader (gruppkonto)"
"a5810","5810","Tickets","expense","l10n_se.account_tag_32","False","Biljetter"
"a5820","5820","Car hire costs","expense","l10n_se.account_tag_32","False","Hyrbilskostnader"
"a5831","5831","Board and lodging in Sweden","expense","l10n_se.account_tag_32","False","Kost och logi i Sverige"
"a5832","5832","Board and lodging abroad","expense","l10n_se.account_tag_32","False","Kost och logi i utlandet"
"a5900","5900","Advertising and PR","expense","l10n_se.account_tag_32","False","Reklam och PR"
"a6071","6071","Entertainment, deductible","expense","l10n_se.account_tag_32","False","Representation, avdragsgill"
"a6072","6072","Entertainment, non-deductible","expense","l10n_se.account_tag_32","False","Representation, ej avdragsgill"
"a6090","6090","Other selling expenses","expense","l10n_se.account_tag_32","False","Övriga försäljningskostnader"
"a6100","6100","Office supplies and printed matter (group account)","expense","l10n_se.account_tag_32","False","Kontorsmateriel och trycksaker (gruppkonto)"
"a6210","6210","Telecommunications","expense","l10n_se.account_tag_32","False","Telekommunikation"
"a6250","6250","Postal services","expense","l10n_se.account_tag_32","False","Postbefordran"
"a6310","6310","Business insurance","expense","l10n_se.account_tag_32","False","Företagsförsäkringar"
"a6350","6350","Losses on trade receivables","expense","l10n_se.account_tag_32","False","Förluster på kundfordringar"
"a6390","6390","Other risk costs","expense","l10n_se.account_tag_32","False","Övriga riskkostnader"
"a6410","6410","Non-salary directors' fees","expense","l10n_se.account_tag_32","False","Styrelsearvoden som inte är lön"
"a6420","6420","Remuneration to the auditor","expense","l10n_se.account_tag_32","False","Ersättningar till revisor"
"a6530","6530","Accounting services","expense","l10n_se.account_tag_32","False","Redovisningstjänster"
"a6540","6540","IT services","expense","l10n_se.account_tag_32","False","IT-tjänster"
"a6550","6550","Consultancy fees","expense","l10n_se.account_tag_32","False","Konsultarvoden"
"a6560","6560","Service fees to professional organisations","expense","l10n_se.account_tag_32","False","Serviceavgifter till branschorganisationer"
"a6570","6570","Bank charges","expense","l10n_se.account_tag_32","False","Bankkostnader"
"a6580","6580","Legal and court costs","expense","l10n_se.account_tag_32","False","Advokat- och rättegångskostnader"
"a6590","6590","Other external services","expense","l10n_se.account_tag_32","False","Övriga externa tjänster"
"a6800","6800","Temporary staff (group account)","expense","l10n_se.account_tag_32","False","Inhyrd personal (gruppkonto)"
"a6970","6970","Newspapers, periodicals and specialised literature","expense","l10n_se.account_tag_32","False","Tidningar, tidskrifter och facklitteratur"
"a6980","6980","Association fees","expense","l10n_se.account_tag_32","False","Föreningsavgifter"
"a6991","6991","Other external costs, deductible","expense","l10n_se.account_tag_32","False","Övriga externa kostnader, avdragsgilla"
"a6992","6992","Other external costs, non-deductible","expense","l10n_se.account_tag_32","False","Övriga externa kostnader, ej avdragsgilla"
"a7010","7010","Wages and salaries to collective employees","expense","l10n_se.account_tag_32","False","Löner till kollektivanställda"
"a7090","7090","Change in holiday pay liability","expense","l10n_se.account_tag_32","False","Förändring av semesterlöneskuld"
"a7210","7210","Wages and salaries to civil servants","expense","l10n_se.account_tag_32","False","Löner till tjänstemän"
"a7220","7220","Wages and salaries of company directors","expense","l10n_se.account_tag_32","False","Löner till företagsledare"
"a7240","7240","Directors' fees","expense","l10n_se.account_tag_32","False","Styrelsearvoden"
"a7290","7290","Change in holiday pay liability","expense","l10n_se.account_tag_32","False","Förändring av semesterlöneskuld"
"a7310","7310","Additional cash payments","expense","l10n_se.account_tag_32","False","Kontanta extraersättningar"
"a7321","7321","Tax-free allowances, Sweden","expense","l10n_se.account_tag_32","False","Skattefria traktamenten, Sverige"
"a7322","7322","Taxable subsistence allowances, Sweden","expense","l10n_se.account_tag_32","False","Skattepliktiga traktamenten, Sverige"
"a7323","7323","Tax-free allowances, abroad","expense","l10n_se.account_tag_32","False","Skattefria traktamenten, utlandet"
"a7324","7324","Taxable daily allowances, abroad","expense","l10n_se.account_tag_32","False","Skattepliktiga traktamenten, utlandet"
"a7331","7331","Tax-free car allowances","expense","l10n_se.account_tag_32","False","Skattefria bilersättningar"
"a7332","7332","Taxable car allowances","expense","l10n_se.account_tag_32","False","Skattepliktiga bilersättningar"
"a7380","7380","Costs of benefits to employees","expense","l10n_se.account_tag_32","False","Kostnader förmåner till anställda"
"a7385","7385","Cost of free car","expense","l10n_se.account_tag_32","False","Kostnader för fri bil"
"a7390","7390","Other reimbursements and benefits","expense","l10n_se.account_tag_32","False","Övriga kostnadsersättningar och förmåner"
"a7410","7410","Pension insurance premiums","expense","l10n_se.account_tag_32","False","Pensionsförsäkringspremier"
"a7490","7490","Other pension costs","expense","l10n_se.account_tag_32","False","Övriga pensionskostnader"
"a7511","7511","Employer's contribution for wages and salaries","expense","l10n_se.account_tag_32","False","Arbetsgivaravgift för löner och ersättningar"
"a7512","7512","Employer's contributions for benefit values","expense","l10n_se.account_tag_32","False","Arbetsgivaravgifter för förmånsvärden"
"a7519","7519","Employer's contributions for holiday and salary liabilities","expense","l10n_se.account_tag_32","False","Arbetsgivaravgifter för semester- och löneskulder"
"a7530","7530","Special payroll tax","expense","l10n_se.account_tag_32","False","Särskild Löneskatt"
"a7550","7550","Tax on returns on pension funds","expense","l10n_se.account_tag_32","False","Avkastningsskatt på pensionsmedel"
"a7570","7570","Premiums for labour market insurance","expense","l10n_se.account_tag_32","False","Premier för arbetsmarknadsförsäkringar"
"a7580","7580","Group insurance premiums","expense","l10n_se.account_tag_32","False","Gruppförsäkringspremier"
"a7590","7590","Other social and other contributions according to law and agreements","expense","l10n_se.account_tag_32","False","Övriga sociala och andra avgifter enligt lag och avtal"
"a7600","7600","Other staff costs (group account)","expense","l10n_se.account_tag_32","False","Övriga personalkostnader (gruppkonto)"
"a7610","7610","Education and training","expense","l10n_se.account_tag_32","False","Utbildning"
"a7621","7621","Medical and health care, deductible","expense","l10n_se.account_tag_32","False","Sjuk- och hälsovård, avdragsgill"
"a7622","7622","Medical and health care, non-deductible","expense","l10n_se.account_tag_32","False","Sjuk- och hälsovård, ej avdragsgill"
"a7631","7631","Staff representation, deductible","expense","l10n_se.account_tag_32","False","Personalrepresentation, avdragsgill"
"a7632","7632","Staff representation, non-deductible","expense","l10n_se.account_tag_32","False","Personalrepresentation, ej avdragsgill"
"a7720","7720","Impairment of land and buildings","expense","l10n_se.account_tag_32","False","Nedskrivningar av byggnader och mark"
"a7730","7730","Impairment of machinery and equipment","expense","l10n_se.account_tag_32","False","Nedskrivningar av maskiner och inventarier"
"a7810","7810","Amortisation of intangible assets","expense_depreciation","l10n_se.account_tag_32","False","Avskrivningar av immateriella anläggningstillgångar"
"a7820","7820","Depreciation of buildings and land improvements","expense_depreciation","l10n_se.account_tag_32","False","Avskrivningar på byggnader och markanläggningar"
"a7830","7830","Depreciation of machinery and equipment","expense_depreciation","l10n_se.account_tag_32","False","Avskrivningar maskiner och inventarier"
"a7970","7970","Loss on disposal of intangible and tangible fixed assets","expense","l10n_se.account_tag_32","False","Förlust vid avyttring av immateriella och materiella anläggningstillgångar"
"a7990","7990","Other operating expenses","expense","l10n_se.account_tag_32","False","Övriga rörelsekostnader"
"a8210","8210","Dividends on shares in other enterprises","income_other","l10n_se.account_tag_33","False","Utdelningar på andelar i andra företag"
"a8220","8220","Profit on sale of securities and long-term receivables from other enterprises","income_other","l10n_se.account_tag_33","False","Resultat vid försäljning av värdepapper i och långfristiga fordringar hos andra företag"
"a8250","8250","Interest income on long-term receivables from and securities held by other enterprises","income_other","l10n_se.account_tag_33","False","Ränteintäkter från långfristiga fordringar hos och värdepapper i andra företag"
"a8270","8270","Impairment losses on holdings of shares in and long-term receivables from other enterprises","expense","l10n_se.account_tag_33","False","Nedskrivningar av innehav av andelar i och långfristiga fordringar hos andra företag"
"a8310","8310","Interest income on current assets","income_other","l10n_se.account_tag_33","False","Ränteintäkter från omsättningstillgångar"
"a8314","8314","Tax-free interest income","income_other","l10n_se.account_tag_33","False","Skattefria ränteintäkter"
"a8330","8330","Exchange rate differences on short-term receivables and investments","income_other","l10n_se.account_tag_33","False","Valutakursdifferenser på kortfristiga fordringar och placeringar"
"a8340","8340","Dividends on short-term investments","income_other","l10n_se.account_tag_33","False","Utdelningar på kortfristiga placeringar"
"a8350","8350","Gain on sale of short-term investments","income_other","l10n_se.account_tag_33","False","Resultat vid försäljning av kortfristiga placeringar"
"a8390","8390","Other financial income","income_other","l10n_se.account_tag_33","False","Övriga finansiella intäkter"
"a8410","8410","Interest expense on long-term liabilities","expense","l10n_se.account_tag_33","False","Räntekostnader för långfristiga skulder"
"a8420","8420","Interest expense on current liabilities","expense","l10n_se.account_tag_33","False","Räntekostnader för kortfristiga skulder"
"a8422","8422","Interest on arrears on trade payables","expense","l10n_se.account_tag_33","False","Dröjsmålsräntor för leverantörsskulder"
"a8423","8423","Interest expense on taxes and duties","expense","l10n_se.account_tag_33","False","Räntekostnader för skatter och avgifter"
"a8430","8430","Exchange rate differences on liabilities","expense","l10n_se.account_tag_33","False","Valutakursdifferenser på skulder"
"a8811","8811","Allocation to tax allocation reserve","expense","l10n_se.account_tag_34","False","Avsättning till periodiseringsfond"
"a8819","8819","Reversal from tax allocation reserve","expense","l10n_se.account_tag_34","False","Återföring från periodiseringsfond"
"a8850","8850","Change in excess depreciation","expense","l10n_se.account_tag_34","False","Förändring av överavskrivningar"
"a8910","8910","Taxes charged to the profit and loss account","expense","l10n_se.account_tag_35","False","Skatt som belastar årets resultat"
"a8990","8990","Profit and loss account","expense","l10n_se.account_tag_36","False","Resultat"
"a8999","8999","Profit for the year","expense","l10n_se.account_tag_36","False","Årets resultat"
"a9993","9993","Cash Discount Loss","expense","l10n_se.account_tag_36","False","Cash Discount Loss"
"a9994","9994","Cash Discount Gain","income_other","l10n_se.account_tag_33","False","Cash Discount Gain"

```

## File: data\template\account.account-se_K2.csv

```csv
"id","code","name","account_type","tag_ids","reconcile","name@sv_SE"
"a1020","1020","Concessions, etc.","asset_non_current","l10n_se.account_tag_1","False","Koncessioner m.m."
"a1028","1028","Accumulated impairment losses on concessions, etc.","asset_non_current","l10n_se.account_tag_1","False","Ackumulerade nedskrivningar på koncessioner m.m."
"a1029","1029","Accumulated amortisation of concessions etc.","asset_non_current","l10n_se.account_tag_1","False","Ackumulerade avskrivningar på koncessioner m.m."
"a1038","1038","Accumulated impairment losses on patents","asset_non_current","l10n_se.account_tag_1","False","Ackumulerade nedskrivningar på patent"
"a1040","1040","Licences","asset_non_current","l10n_se.account_tag_1","False","Licenser"
"a1048","1048","Accumulated impairment losses on licences","asset_non_current","l10n_se.account_tag_1","False","Ackumulerade nedskrivningar på licenser"
"a1049","1049","Accumulated amortisation of licences","asset_non_current","l10n_se.account_tag_1","False","Ackumulerade avskrivningar på licenser"
"a1050","1050","Trade marks","asset_non_current","l10n_se.account_tag_1","False","Varumärken"
"a1058","1058","Accumulated impairment losses on trademarks","asset_non_current","l10n_se.account_tag_1","False","Ackumulerade nedskrivningar på varumärken"
"a1059","1059","Accumulated amortisation of trademarks","asset_non_current","l10n_se.account_tag_1","False","Ackumulerade avskrivningar på varumärken"
"a1060","1060","Tenancies, leaseholds and similar","asset_non_current","l10n_se.account_tag_1","False","Hyresrätter, tomträtter och liknande"
"a1068","1068","Accumulated impairment losses on leasehold rights, freehold rights and similar","asset_non_current","l10n_se.account_tag_1","False","Ackumulerade nedskrivningar på hyresrätter, tomträtter och liknande"
"a1070","1070","Goodwill","asset_non_current","l10n_se.account_tag_1","False","Goodwill"
"a1078","1078","Accumulated amortisation of goodwill","asset_non_current","l10n_se.account_tag_1","False","Ackumulerade nedskrivningar på goodwill"
"a1079","1079","Accumulated amortisation of goodwill","asset_non_current","l10n_se.account_tag_1","False","Ackumulerade avskrivningar på goodwill"
"a1080","1080","Advances for intangible fixed assets","asset_non_current","l10n_se.account_tag_1","False","Förskott för immateriella anläggningstillgångar"
"a1111","1111","Buildings on other people's land","asset_fixed","l10n_se.account_tag_2","False","Byggnader på annans mark"
"a1112","1112","Buildings on own land","asset_fixed","l10n_se.account_tag_2","False","Byggnader på egen mark"
"a1118","1118","Accumulated impairment losses on buildings","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade nedskrivningar på byggnader"
"a1120","1120","Improvements to property owned by others","asset_fixed","l10n_se.account_tag_2","False","Förbättringsutgifter på annans fastighet"
"a1129","1129","Accumulated depreciation of improvements on other people's property","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade avskrivningar på förbättringsutgifter på annans fastighet"
"a1140","1140","Land and undeveloped land","asset_fixed","l10n_se.account_tag_2","False","Tomter och obebyggda markområden"
"a1158","1158","Accumulated impairment losses on land improvements","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade nedskrivningar på markanläggningar"
"a1180","1180","Construction, extension and renovation in progress","asset_fixed","l10n_se.account_tag_2","False","Pågående ny-,till- och ombyggnad"
"a1188","1188","Advances for buildings and land","asset_fixed","l10n_se.account_tag_2","False","Förskott för byggnader och mark"
"a1211","1211","Machinery","asset_fixed","l10n_se.account_tag_2","False","Maskiner"
"a1213","1213","Other technical installations","asset_fixed","l10n_se.account_tag_2","False","Andra tekniska anläggningar"
"a1218","1218","Accumulated depreciation of machinery and other technical equipment","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade nedskrivningar på maskiner och andra tekniska anläggningar"
"a1221","1221","Equipment","asset_fixed","l10n_se.account_tag_2","False","Inventarier"
"a1222","1222","Building inventory","asset_fixed","l10n_se.account_tag_2","False","Byggnadsinventarier"
"a1223","1223","Land equipment","asset_fixed","l10n_se.account_tag_2","False","Markinventarier"
"a1225","1225","Tools","asset_fixed","l10n_se.account_tag_2","False","Verktyg"
"a1228","1228","Accumulated impairment losses on equipment and tools","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade nedskrivningar på inventarier och verktyg"
"a1229","1229","Accumulated depreciation of equipment and tools","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade avskrivningar på inventarier och verktyg"
"a1230","1230","Installations","asset_fixed","l10n_se.account_tag_2","False","Installationer"
"a1231","1231","Installations on own property","asset_fixed","l10n_se.account_tag_2","False","Installationer på egen fastighet"
"a1232","1232","Installations on another's property","asset_fixed","l10n_se.account_tag_2","False","Installationer på annans fastighet"
"a1238","1238","Accumulated depreciation of installations","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade nedskrivningar på installationer"
"a1239","1239","Accumulated depreciation of installations","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade avskrivningar på installationer"
"a1240","1240","Cars and other means of transport","asset_fixed","l10n_se.account_tag_2","False","Bilar och andra transportmedel"
"a1241","1241","Passenger cars","asset_fixed","l10n_se.account_tag_2","False","Personbilar"
"a1242","1242","lorries","asset_fixed","l10n_se.account_tag_2","False","Lastbilar"
"a1243","1243","Trucks","asset_fixed","l10n_se.account_tag_2","False","Truckar"
"a1244","1244","Working machines","asset_fixed","l10n_se.account_tag_2","False","Arbetsmaskiner"
"a1245","1245","Tractors","asset_fixed","l10n_se.account_tag_2","False","Traktorer"
"a1246","1246","Motorcycles, mopeds and scooters","asset_fixed","l10n_se.account_tag_2","False","Motorcyklar, mopeder och skotrar"
"a1247","1247","Boats, aeroplanes and helicopters","asset_fixed","l10n_se.account_tag_2","False","Båtar, flygplan och helikoptrar"
"a1248","1248","Accumulated impairment losses on cars and other means of transport","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade nedskrivningar på bilar och andra transportmedel"
"a1251","1251","Computers, business","asset_fixed","l10n_se.account_tag_2","False","Datorer, företaget"
"a1257","1257","Computers, personnel","asset_fixed","l10n_se.account_tag_2","False","Datorer, personal"
"a1258","1258","Accumulated impairment losses on computers","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade nedskrivningar på datorer"
"a1280","1280","Construction in progress and advance payments for machinery and equipment","asset_fixed","l10n_se.account_tag_2","False","Pågående nyanläggningar och förskott för maskiner och inventarier"
"a1281","1281","Construction in progress, machinery and equipment","asset_fixed","l10n_se.account_tag_2","False","Pågående nyanläggningar, maskiner och inventarier"
"a1288","1288","Advances for machinery and equipment","asset_fixed","l10n_se.account_tag_2","False","Förskott för maskiner och inventarier"
"a1292","1292","Animals classified as fixed assets","asset_fixed","l10n_se.account_tag_2","False","Djur som klassificeras som anläggningstillgång"
"a1298","1298","Accumulated impairment losses on other tangible fixed assets","asset_fixed","l10n_se.account_tag_2","False","Ackumulerade nedskrivningar på övriga materiella anläggningstillgångar"
"a1310","1310","Shares in group companies","asset_non_current","l10n_se.account_tag_3","False","Andelar i koncernföretag"
"a1311","1311","Shares in listed Swedish group companies","asset_non_current","l10n_se.account_tag_3","False","Aktier i noterade svenska koncernföretag"
"a1312","1312","Shares in unlisted Swedish group companies","asset_non_current","l10n_se.account_tag_3","False","Aktier i onoterade svenska koncernföretag"
"a1313","1313","Shares in listed foreign group companies","asset_non_current","l10n_se.account_tag_3","False","Aktier i noterade ütlandska koncernföretag"
"a1314","1314","Shares in unlisted foreign group companies","asset_non_current","l10n_se.account_tag_3","False","Aktier i onoterade ütlandska koncernföretag"
"a1316","1316","Other shares in Swedish group companies","asset_non_current","l10n_se.account_tag_3","False","Andra andelar i svenska koncernföretag"
"a1317","1317","Other shares in Dutch group companies","asset_non_current","l10n_se.account_tag_3","False","Andra andelar i ütlandska koncernföretag"
"a1318","1318","Accumulated impairment losses on shares in group enterprises","asset_non_current","l10n_se.account_tag_3","False","Ackumulerade nedskrivningar av andelar i koncernföretag"
"a1320","1320","Long-term receivables from group companies","asset_non_current","l10n_se.account_tag_3","False","Långfristiga fordringar hos koncernföretag"
"a1321","1321","Long-term receivables from parent companies","asset_non_current","l10n_se.account_tag_3","False","Långfristiga fordringar hos moderföretag"
"a1322","1322","Long-term receivables from subsidiaries","asset_non_current","l10n_se.account_tag_3","False","Långfristiga fordringar hos dotterföretag"
"a1323","1323","Long-term receivables from other group companies","asset_non_current","l10n_se.account_tag_3","False","Långfristiga fordringar hos andra koncernföretag"
"a1328","1328","Accumulated impairment of long-term receivables from group companies","asset_non_current","l10n_se.account_tag_3","False","Ackumulerade nedskrivningar av långfristiga fordringar hos koncernföretag"
"a1330","1330","Investments in associates, jointly controlled entities and other entities in which there is an ownership interest","asset_non_current","l10n_se.account_tag_3","False","Andelar i intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"a1331","1331","Shares in associates","asset_non_current","l10n_se.account_tag_3","False","Andelar i intresseföretag"
"a1332","1332","Accumulated impairment losses on investments in associates","asset_non_current","l10n_se.account_tag_3","False","Ackumulerade nedskrivningar av andelar i intresseföretag"
"a1333","1333","Shares in jointly controlled entities","asset_non_current","l10n_se.account_tag_3","False","Andelar i gemensamt styrda företag"
"a1334","1334","Accumulated impairment of investments in jointly controlled entities","asset_non_current","l10n_se.account_tag_3","False","Ackumulerade nedskrivningar av andelar i gemensamt styrda företag"
"a1336","1336","Shares in other enterprises in which there is a participating interest","asset_non_current","l10n_se.account_tag_3","False","Andelar i övriga företag som det finns ett ägarintresse i"
"a1337","1337","Accumulated impairment losses on participations in other enterprises in which there is a participating interest","asset_non_current","l10n_se.account_tag_3","False","Ackumulerade nedskrivningar av andelar i övriga företag som det finns ett ägarintresse i"
"a1338","1338","Accumulated impairment losses on investments in associates, jointly controlled entities and other entities in which an ownership interest exists","asset_non_current","l10n_se.account_tag_3","False","Ackumulerade nedskrivningar av andelar i intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"a1340","1340","Non-current receivables from associates, jointly controlled entities and other entities in which there is a participating interest","asset_non_current","l10n_se.account_tag_3","True","Långfristiga fordringar hos intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"a1341","1341","Long-term receivables from associates","asset_non_current","l10n_se.account_tag_3","True","Långfristiga fordringar hos intresseföretag"
"a1342","1342","Accumulated impairment losses on long-term receivables from associates","asset_non_current","l10n_se.account_tag_3","True","Ackumulerade nedskrivningar av långfristiga fordringar hos intresseföretag"
"a1343","1343","Long-term receivables from jointly controlled entities","asset_non_current","l10n_se.account_tag_3","True","Långfristiga fordringar hos gemensamt styrda företag"
"a1344","1344","Accumulated impairment losses on long-term receivables from jointly controlled entities","asset_non_current","l10n_se.account_tag_3","True","Ackumulerade nedskrivningar av långfristiga fordringar hos gemensamt styrda företag"
"a1346","1346","Long-term receivables from other entities in which there is an ownership interest","asset_non_current","l10n_se.account_tag_3","True","Långfristiga fordringar hos övriga företag som det finns ett ägarintresse i"
"a1347","1347","Accumulated impairment losses on long-term receivables from other investee companies","asset_non_current","l10n_se.account_tag_3","True","Ackumulerade nedskrivningar av långfristiga fordringar hos övriga företag som det finns ett ägarintresse i"
"a1348","1348","Accumulated impairment losses on non-current receivables from associates, jointly controlled entities and other entities in which there is an ownership interest.","asset_non_current","l10n_se.account_tag_3","True","Ackumulerade nedskrivningar av långfristiga fordringar hos intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"a1351","1351","Shares in listed companies","asset_non_current","l10n_se.account_tag_3","False","Andelar i noterade företag"
"a1352","1352","Other shares","asset_non_current","l10n_se.account_tag_3","False","Andra andelar"
"a1353","1353","Shares in housing associations","asset_non_current","l10n_se.account_tag_3","False","Andelar i bostadsrättsföreningar"
"a1354","1354","Bonds and notes","asset_non_current","l10n_se.account_tag_3","False","Obligationer"
"a1356","1356","Shares in economic associations, other corporations","asset_non_current","l10n_se.account_tag_3","False","Andelar i ekonomiska föreningar, övriga företag"
"a1357","1357","Shares in partnerships, other companies","asset_non_current","l10n_se.account_tag_3","False","Andelar i handelsbolag, andra företag"
"a1358","1358","Accumulated impairments of other shares and securities","asset_non_current","l10n_se.account_tag_3","False","Ackumulerade nedskrivningar av andra andelar och värdepapper"
"a1360","1360","Accumulated impairment of loans to partners or related parties according to ABL, long-term part","asset_non_current","l10n_se.account_tag_3","False","Ackumulerade nedskrivningar av lån till delägare eller närstående enligt ABL, långfristig del"
"a1369","1369","Loans to partners or related parties according to ABL, long-term part","asset_non_current","l10n_se.account_tag_3","False","Lån till delägare eller närstående enligt ABL, långfristig del"
"a1381","1381","Long-term promissory note receivables","asset_non_current","l10n_se.account_tag_3","False","Långfristiga reversfordringar"
"a1382","1382","Long-term receivables from employees","asset_non_current","l10n_se.account_tag_3","False","Långfristiga fordringar hos anställda"
"a1383","1383","Deposits paid, long-term","asset_non_current","l10n_se.account_tag_3","False","Lämnade depositioner, långfristiga"
"a1384","1384","Derivatives","asset_non_current","l10n_se.account_tag_3","False","Derivat"
"a1385","1385","Value of endowment insurance","asset_non_current","l10n_se.account_tag_3","False","Värde av kapitalförsäkring"
"a1387","1387","Long-term contractual receivables","asset_non_current","l10n_se.account_tag_3","False","Långfristiga kontraktsfordringar"
"a1388","1388","Long-term trade receivables","asset_non_current","l10n_se.account_tag_3","False","Långfristiga kundfordringar"
"a1389","1389","Accumulated impairment of other long-term receivables","asset_non_current","l10n_se.account_tag_3","False","Ackumulerade nedskrivningar av andra långfristiga fordringar"
"a1420","1420","Stocks of consumables and supplies","asset_current","l10n_se.account_tag_4","False","Lager av tillsatsmaterial och förnödenheter"
"a1429","1429","Change in stocks of consumables and supplies","asset_current","l10n_se.account_tag_4","False","Förändring av lager av tillsatsmaterial och förnödenheter"
"a1465","1465","Stocks of goods purchased for resale","asset_current","l10n_se.account_tag_4","False","Lager av varor VMB"
"a1466","1466","Impairment of inventories of intermediate goods","asset_current","l10n_se.account_tag_4","False","Nedskrivning av varor VMB"
"a1467","1467","Stocks of goods in progress simplified","asset_current","l10n_se.account_tag_4","False","Lager av varor VMB förenklad"
"a1471","1471","Work in progress, costs incurred","asset_current","l10n_se.account_tag_4","False","Pågående arbeten, nedlagda kostnader"
"a1478","1478","Work in progress, invoicing","asset_current","l10n_se.account_tag_4","False","Pågående arbeten, fakturering"
"a1480","1480","Advances for goods and services","asset_prepayments","l10n_se.account_tag_4","False","Förskott för varor och tjänster"
"a1481","1481","Letters of credit","asset_prepayments","l10n_se.account_tag_4","False","Remburser"
"a1489","1489","Other advances to suppliers","asset_prepayments","l10n_se.account_tag_4","False","Övriga förskott till leverantörer"
"a1491","1491","Stocks of securities","asset_prepayments","l10n_se.account_tag_4","False","Lager av värdepapper"
"a1492","1492","Stocks of real estate","asset_prepayments","l10n_se.account_tag_4","False","Lager av fastigheter"
"a1493","1493","Animals classified as current assets","asset_prepayments","l10n_se.account_tag_4","False","Djur som klassificeras som omsättningstillgång"
"a1511","1511","Trade receivables","asset_receivable","l10n_se.account_tag_5","True","Kundfordringar"
"a1512","1512","Mortgaged accounts receivable (factoring)","asset_receivable","l10n_se.account_tag_5","True","Belånade kundfordringar (factoring)"
"a1516","1516","Disputed trade receivables","asset_receivable","l10n_se.account_tag_5","True","Tvistiga kundfordringar"
"a1518","1518","Non-accrual accounts receivable","asset_receivable","l10n_se.account_tag_5","True","Ej reskontrafördra kundfordringar"
"a1520","1520","Bills of exchange receivable","asset_receivable","l10n_se.account_tag_5","True","Växelfordringar"
"a1525","1525","Doubtful bills receivable","asset_receivable","l10n_se.account_tag_5","True","Osäkra växelfordringar"
"a1529","1529","Impairment of trade receivables","asset_receivable","l10n_se.account_tag_5","True","Nedskrivning av växelfordringar"
"a1530","1530","Contractual receivables","asset_receivable","l10n_se.account_tag_5","True","Kontraktsfordringar"
"a1531","1531","Contract receivables","asset_receivable","l10n_se.account_tag_5","True","Kontraktsfordringar"
"a1532","1532","Encumbered contract receivables","asset_receivable","l10n_se.account_tag_5","True","Belånade kontraktsfordringar"
"a1536","1536","Disputed contract receivables","asset_receivable","l10n_se.account_tag_5","True","Tvistiga kontraktsfordringar"
"a1539","1539","Impairment of contract receivables","asset_receivable","l10n_se.account_tag_5","True","Nedskrivning av kontraktsfordringar"
"a1550","1550","Consignment receivables","asset_receivable","l10n_se.account_tag_5","True","Konsignationsfordringar"
"a1560","1560","Trade receivables from group companies","asset_receivable","l10n_se.account_tag_5","True","Kundfordringar hos koncernföretag"
"a1561","1561","Accounts receivable from parent company","asset_receivable","l10n_se.account_tag_5","True","Kundfordringar hos moderföretag"
"a1562","1562","Accounts receivable from subsidiaries","asset_receivable","l10n_se.account_tag_5","True","Kundfordringar hos dotterföretag"
"a1563","1563","Accounts receivable from other group companies","asset_receivable","l10n_se.account_tag_5","True","Kundfordringar hos andra koncernföretag"
"a1568","1568","Non-recognition of accounts receivable from group companies","asset_receivable","l10n_se.account_tag_5","True","Ej reskontrafördra kundfordringar hos koncernföretag"
"a1569","1569","Impairment of trade receivables from group companies","asset_receivable","l10n_se.account_tag_5","True","Nedskrivning av kundfordringar hos koncernföretag"
"a1570","1570","Accounts receivable from associates, jointly controlled entities and other entities in which there is an ownership interest.","asset_receivable","l10n_se.account_tag_5","True","Kundfordringar hos intresseföretag, gemensamt styrda företag och övriga företag som det finns et ägarintresse i"
"a1571","1571","Accounts receivable from associates","asset_receivable","l10n_se.account_tag_5","True","Kundfordringar hos intresseföretag"
"a1572","1572","Trade receivables from jointly controlled entities","asset_receivable","l10n_se.account_tag_5","True","Kundfordringar hos gemensamt styrda företag"
"a1573","1573","Accounts receivable from other enterprises in which there is an ownership interest","asset_receivable","l10n_se.account_tag_5","True","Kundfordringar hos övriga företag som det finns ett ägarintresse i"
"a1611","1611","Travel advances","asset_receivable","l10n_se.account_tag_5","True","Reseförskott"
"a1612","1612","Cash advances","asset_receivable","l10n_se.account_tag_5","True","Kassaförskott"
"a1613","1613","Other advances","asset_receivable","l10n_se.account_tag_5","True","Övriga förskott"
"a1614","1614","Temporary loans to employees","asset_receivable","l10n_se.account_tag_5","True","Tillfälliga lån till anställda"
"a1619","1619","Other receivables from employees","asset_receivable","l10n_se.account_tag_5","True","Övriga fordringar hos anställda"
"a1620","1620","Accrued but uninvoiced income","asset_receivable","l10n_se.account_tag_5","True","Upparbetad men ej fakturerad intäkt"
"a1660","1660","Current receivables from group companies","asset_current","l10n_se.account_tag_5","False","Kortfristiga fordringar hos koncernföretag"
"a1661","1661","Current receivables from parent company","asset_current","l10n_se.account_tag_5","False","Kortfristiga fordringar hos moderföretag"
"a1662","1662","Current receivables from subsidiaries","asset_current","l10n_se.account_tag_5","False","Kortfristiga fordringar hos dotterföretag"
"a1663","1663","Current receivables from other group companies","asset_current","l10n_se.account_tag_5","False","Kortfristiga fordringar hos andra koncernföretag"
"a1670","1670","Current receivables from associates, jointly controlled entities and other entities in which there is an ownership interest.","asset_current","l10n_se.account_tag_5","False","Kortfristiga fordringar hos intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"a1671","1671","Current receivables from associates","asset_current","l10n_se.account_tag_5","False","Kortfristiga fordringar hos intresseföretag"
"a1672","1672","Current receivables from jointly controlled entities","asset_current","l10n_se.account_tag_5","False","Kortfristiga fordringar hos gemensamt styrda företag"
"a1673","1673","Current receivables from other entities in which there is an ownership interest","asset_current","l10n_se.account_tag_5","False","Kortfristiga fordringar hos övriga företag som det finns ett ägarintresse i"
"a1681","1681","Outlays for customers","asset_current","l10n_se.account_tag_5","False","Utlägg för kunder"
"a1682","1682","Short-term loan receivables","asset_current","l10n_se.account_tag_5","False","Kortfristiga lånefordringar"
"a1683","1683","Derivatives","asset_current","l10n_se.account_tag_5","False","Derivat"
"a1684","1684","Current receivables from suppliers","asset_current","l10n_se.account_tag_5","False","Kortfristiga fordringar hos leverantörer"
"a1685","1685","Current receivables from partners or related parties","asset_current","l10n_se.account_tag_5","False","Kortfristiga fordringar hos delägare eller närstående"
"a1687","1687","Current portion of long-term receivables","asset_current","l10n_se.account_tag_5","False","Kortfristiga del av långfristiga fordringar"
"a1688","1688","Labour market insurance claim","asset_current","l10n_se.account_tag_5","False","Fordran arbetsmarknadsförsäkringar"
"a1689","1689","Other current receivables","asset_current","l10n_se.account_tag_5","False","Övriga kortfristiga fordringar"
"a1690","1690","Receivables for subscribed but unpaid share capital","asset_receivable","l10n_se.account_tag_0","True","Fordringar för tecknat men ej inbetalt aktiekapital"
"a1770","1770","Assets of a cost nature","asset_current","l10n_se.account_tag_5","False","Tillgångar av kostnadsnatur"
"a1780","1780","Accrued contractual income","asset_current","l10n_se.account_tag_5","False","Upplupna avtalsintäkter"
"a1820","1820","Bonds and notes","asset_current","l10n_se.account_tag_6","False","Obligationer"
"a1830","1830","Convertible debt securities","asset_current","l10n_se.account_tag_6","False","Konvertibla skuldebrev"
"a1860","1860","Shares in group companies, short-term","asset_current","l10n_se.account_tag_6","False","Andelar i koncernföretag, kortfristigt"
"a1886","1886","Derivatives","asset_current","l10n_se.account_tag_6","False","Derivat"
"a1889","1889","Shares in other companies","asset_current","l10n_se.account_tag_6","False","Andelar i övriga företag"
"a1911","1911","Principal cash","asset_cash","l10n_se.account_tag_7","False","Huvudkassa"
"a1912","1912","Cash 2","asset_cash","l10n_se.account_tag_7","False","Kassa 2"
"a1913","1913","Cash 3","asset_cash","l10n_se.account_tag_7","False","Kassa 3"
"a1950","1950","Certificates of deposit","asset_cash","l10n_se.account_tag_7","False","Bankcertifikat"
"a1960","1960","Group account parent company","asset_cash","l10n_se.account_tag_7","False","Koncernkonto moderföretag"
"a1970","1970","Special bank accounts","asset_cash","l10n_se.account_tag_7","False","Särskilda bankkonton"
"a1972","1972","Author's account","asset_cash","l10n_se.account_tag_7","False","Upphovsmannakonto"
"a1973","1973","Forestry account","asset_cash","l10n_se.account_tag_7","False","Skogskonto"
"a1974","1974","Blocked bank funds","asset_cash","l10n_se.account_tag_7","False","Spärrade bankmedel"
"a1979","1979","Special bank accounts","asset_cash","l10n_se.account_tag_7","False","Särskilda bankkonton"
"a1980","1980","Currency accounts","asset_cash","l10n_se.account_tag_7","False","Valutakonton"
"a1990","1990","Accounting funds","asset_cash","l10n_se.account_tag_7","False","Redovisningsmedel"
"a2050","2050","Allocation to expansion fund","equity","l10n_se.account_tag_19","False","Avsättning till expansionsfond"
"a2061","2061","Equity capital / endowment capital / core capital","equity","l10n_se.account_tag_13","False","Eget kapital / stiftelsekapital / grundkapital"
"a2065","2065","Change in fair value fund","equity","l10n_se.account_tag_16","False","Förändring i fond för verkligt värde"
"a2066","2066","Hedge fund","equity","l10n_se.account_tag_11","False","Värdesäkringsfond"
"a2067","2067","Profit or loss brought forward / Capital brought forward","equity","l10n_se.account_tag_12","False","Balanserad vinst eller förlust / Balanserad kapital"
"a2068","2068","Profit or loss from previous year","equity","l10n_se.account_tag_12","False","Vinst eller förlust fran föregående år"
"a2069","2069","Profit or loss for the year","equity","l10n_se.account_tag_17","False","Årets resultat"
"a2071","2071","Purpose 1","equity","l10n_se.account_tag_37","False","Ändamål 1"
"a2072","2072","Purpose 2","equity","l10n_se.account_tag_37","False","Ändamål 2"
"a2080","2080","Restricted equity","equity","l10n_se.account_tag_14","False","Bundet eget kapital"
"a2082","2082","Unregistered share capital","equity","l10n_se.account_tag_15","False","Ej registrerat aktiekapital"
"a2084","2084","Subscriptions","equity","l10n_se.account_tag_40","False","Förlagsinsatser"
"a2085","2085","Revaluation reserve","equity","l10n_se.account_tag_10","False","Uppskrivningsfond"
"a2087","2087","Restricted share premium account / Rights issue","equity","l10n_se.account_tag_9","False","Bunden överkursfond / Insatsemission"
"a2088","2088","External maintenance fund","equity","l10n_se.account_tag_11","False","Fond för yttre underhåll"
"a2089","2089","Development expenditure fund","equity","l10n_se.account_tag_11","False","Fund för utvecklingsutgifter"
"a2093","2093","Receiving shareholder contributions","equity","l10n_se.account_tag_40","False","Erhålla aktieägartillskott"
"a2094","2094","Own shares","equity","l10n_se.account_tag_40","False","Egna aktier"
"a2095","2095","Merger result","equity","l10n_se.account_tag_39","False","Fusionsresultat"
"a2097","2097","Unrestricted share premium account","equity","l10n_se.account_tag_9","False","Fri överkursfond"
"a2110","2110","Accrual fund","equity","l10n_se.account_tag_18","False","Periodiseringsfond"
"a2130","2130","Tax allocation reserve 2020 - No. 2","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2020 - nr 2"
"a2131","2131","Tax allocation fund 2021 - no. 2","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2021 - nr 2"
"a2132","2132","Accrual fund 2022 - No. 2","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2022 - nr 2"
"a2133","2133","Tax allocation fund 2023 - No 2","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2023 - nr 2"
"a2134","2134","Accrual fund 2024 - No 2","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2024 - nr 2"
"a2135","2135","Accrual fund 2015 - No 2","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2015 - nr 2"
"a2136","2136","Accrual fund 2016 - No 2","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2016 - nr 2"
"a2137","2137","Accrual fund 2017 - No 2","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2017 - nr 2"
"a2138","2138","Accrual fund 2018 - No 2","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2018 - nr 2"
"a2139","2139","Accrual fund 2019 - No 2","equity","l10n_se.account_tag_18","False","Periodiseringsfond 2019 - nr 2"
"a2151","2151","Accumulated excess amortisation of intangible fixed assets","liability_non_current","l10n_se.account_tag_18","False","Ackumulerade överavskrivningar på immateriella anläggningstillgångar"
"a2152","2152","Accumulated excess depreciation of buildings and land improvements","liability_non_current","l10n_se.account_tag_18","False","Ackumulerade överavskrivningar på byggnader lch markanläggningar"
"a2153","2153","Accumulated excess depreciation of machinery and equipment","liability_non_current","l10n_se.account_tag_18","False","Ackumulerade överavskrivningar på maskiner och inventarier"
"a2160","2160","Compensation fund","liability_non_current","l10n_se.account_tag_18","False","Ersättningsfond"
"a2161","2161","Compensation fund for machinery and equipment","liability_non_current","l10n_se.account_tag_18","False","Ersättningsfond maskiner och inventarier"
"a2162","2162","Compensation fund for buildings and land improvements","liability_non_current","l10n_se.account_tag_18","False","Ersättningsfond byggnader och markanläggningar"
"a2164","2164","Compensation fund for animal stocks in agriculture and reindeer husbandry","liability_non_current","l10n_se.account_tag_18","False","Ersättningsfond for djurlager i jordbruk och renskötsel"
"a2190","2190","Other untaxed reserves","liability_non_current","l10n_se.account_tag_18","False","Övriga obeskattade reserver"
"a2196","2196","Stock reserve","liability_non_current","l10n_se.account_tag_18","False","Lagerreserv"
"a2199","2199","Other untaxed reserves","liability_non_current","l10n_se.account_tag_18","False","Övriga obeskattade reserver"
"a2230","2230","Other provisions for pensions and similar obligations","liability_non_current","l10n_se.account_tag_19","False","Övriga avsättningar för pensioner och liknande förpliktelser"
"a2250","2250","Other provisions for taxes","liability_non_current","l10n_se.account_tag_19","False","Övriga avsättningar för skatter"
"a2252","2252","Provisions for disputed taxes","liability_non_current","l10n_se.account_tag_19","False","Avsättningar för tvistiga skatter"
"a2253","2253","Provisions for special payroll tax declaration item","liability_non_current","l10n_se.account_tag_19","False","Avsättningar särskild löneskatt deklarationspost"
"a2290","2290","Other provisions","liability_non_current","l10n_se.account_tag_19","False","Övriga avsättningar"
"a2310","2310","Bonds and debentures","liability_non_current","l10n_se.account_tag_20","False","Obligations och förlagslån"
"a2320","2320","Convertible loans and similar","liability_non_current","l10n_se.account_tag_20","False","Konvertibla lån och liknade"
"a2321","2321","Convertible loans","liability_non_current","l10n_se.account_tag_20","False","Konvertibla lån"
"a2322","2322","Loans with options","liability_non_current","l10n_se.account_tag_20","False","Lån förenade med optionsrätt"
"a2323","2323","Profit-sharing loans","liability_non_current","l10n_se.account_tag_20","False","Vinstandelslån"
"a2324","2324","Equity loans","liability_non_current","l10n_se.account_tag_20","False","Kapitalandelslån"
"a2331","2331","Utilised overdraft facility 1","liability_non_current","l10n_se.account_tag_21","False","Utnyttjad checkräkningskredit 1"
"a2332","2332","Utilised overdraft facility 2","liability_non_current","l10n_se.account_tag_21","False","Utnyttjad checkräkningskredit 2"
"a2335","2335","Authorised overdraft facility 1","liability_non_current","l10n_se.account_tag_21","False","Beviljad checkräkningskredit 1"
"a2336","2336","Authorised overdraft facility 2","liability_non_current","l10n_se.account_tag_21","False","Beviljad checkräkningskredit 2"
"a2340","2340","Building credit","liability_non_current","l10n_se.account_tag_21","False","Byggnadskreditiv"
"a2351","2351","Property loans, long-term part","liability_non_current","l10n_se.account_tag_21","False","Fastighetslån, långfristig del"
"a2355","2355","Long-term loans in foreign currency from credit institutions","liability_non_current","l10n_se.account_tag_21","False","Långfristiga lån i utländsk valuta från kreditinstitut"
"a2359","2359","Other long-term loans from credit institutions","liability_non_current","l10n_se.account_tag_21","False","Övriga långfristiga lån från kreditinstitut"
"a2360","2360","Long-term liabilities to group companies","liability_non_current","l10n_se.account_tag_24","False","Långfristiga skulder till koncernföretag"
"a2361","2361","Long-term liabilities to parent companies","liability_non_current","l10n_se.account_tag_24","False","Långfristiga skulder till moderföretag"
"a2362","2362","Long-term liabilities to subsidiaries","liability_non_current","l10n_se.account_tag_24","False","Långfristiga skulder till dotterföretag"
"a2363","2363","Long-term liabilities to other group entities","liability_non_current","l10n_se.account_tag_24","False","Långfristiga skulder till andra koncernföretag"
"a2370","2370","Long-term liabilities to associates, jointly controlled entities and other entities in which an ownership interest exists","liability_non_current","l10n_se.account_tag_26","False","Långfristiga skulder till intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"a2371","2371","Long-term liabilities to associates","liability_non_current","l10n_se.account_tag_26","False","Långfristiga skulder till intresseföretag"
"a2372","2372","Long-term liabilities to jointly controlled entities","liability_non_current","l10n_se.account_tag_26","False","Långfristiga skulder till gemensamt styrda företag"
"a2373","2373","Non-current liabilities to associates other enterprises in which there is a participating interest.","liability_non_current","l10n_se.account_tag_27","False","Långfristiga skulder till intresseföretag övriga företag som det finns ett ägarintresse i"
"a2391","2391","Instalment contracts, long-term part","liability_non_current","l10n_se.account_tag_29","False","Avbetalningskontrakt, långfristiga del"
"a2392","2392","Contingent long-term liabilities","liability_non_current","l10n_se.account_tag_29","False","Villkorliga långfristiga skulder"
"a2394","2394","Long-term supplier credits","liability_non_current","l10n_se.account_tag_29","False","Långfristiga leverantörskrediter"
"a2395","2395","Other long-term loans in foreign currency","liability_non_current","l10n_se.account_tag_29","False","Andra långfristiga lån i utländsk valuta"
"a2396","2396","Derivatives","liability_non_current","l10n_se.account_tag_29","False","Derivat"
"a2397","2397","Deposits received, long-term","liability_non_current","l10n_se.account_tag_29","False","Mottagna depositioner, långfristiga"
"a2399","2399","Other long-term liabilities","liability_non_current","l10n_se.account_tag_29","False","Övriga långfristiga skulder"
"a2411","2411","Short-term loans from credit institutions","liability_current","l10n_se.account_tag_21","False","Kortfristiga lån från kreditinstitut"
"a2412","2412","Building loans, current portion","liability_current","l10n_se.account_tag_21","False","Byggnadskreditiv, kortfristig del"
"a2417","2417","Current portion of long-term liabilities to credit institutions","liability_current","l10n_se.account_tag_21","False","Kortfristiga del av långfristiga skulder till kreditinstitut"
"a2419","2419","Other current liabilities to credit institutions","liability_current","l10n_se.account_tag_21","False","Övriga kortfristiga skulder till kreditinstitut"
"a2421","2421","Unredeemed gift cards","liability_current","l10n_se.account_tag_22","False","Ej inlösta presentkort"
"a2429","2429","Other advances from customers","liability_current","l10n_se.account_tag_22","False","Övriga förskott från kunder"
"a2430","2430","Work in progress","liability_current","l10n_se.account_tag_4","False","Pågående arbeten"
"a2431","2431","Work in progress, invoicing","liability_current","l10n_se.account_tag_4","True","Pågående arbeten, fakturering"
"a2438","2438","Work in progress, costs incurred","liability_current","l10n_se.account_tag_4","False","Pågående arbeten, nedlagda kostnader"
"a2439","2439","Estimated change in work in progress","liability_current","l10n_se.account_tag_4","False","Beräknad förändring av pågående arbeten"
"a2441","2441","Trade payables","liability_payable","l10n_se.account_tag_23","True","Leverantörsskulder"
"a2443","2443","Consignment liabilities","liability_payable","l10n_se.account_tag_23","True","Konsignationsskulder"
"a2445","2445","Disputed accounts payable","liability_payable","l10n_se.account_tag_23","True","Tvistiga leverantörsskulder"
"a2448","2448","Unrecognised trade payables","liability_payable","l10n_se.account_tag_23","True","Ej reskontrafördra leverantörsskulder"
"a2450","2450","Invoiced but not recognised revenue","liability_current","l10n_se.account_tag_23","False","Fakturerad men ej upparbetad intäkt"
"a2460","2460","Accounts payable to group companies","liability_current","l10n_se.account_tag_24","False","Leverantörsskulder till koncernföretag"
"a2461","2461","Accounts payable to parent company","liability_current","l10n_se.account_tag_24","False","Leverantörsskulder till moderföretag"
"a2462","2462","Accounts payable to subsidiaries","liability_current","l10n_se.account_tag_24","False","Leverantörsskulder till dotterföretag"
"a2463","2463","Accounts payable to other group companies","liability_current","l10n_se.account_tag_24","False","Leverantörsskulder till andra koncernföretag"
"a2470","2470","Trade payables to associates, jointly controlled entities and other entities in which there is an ownership interest.","liability_current","l10n_se.account_tag_26","False","Leverantörsskulder till intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"a2471","2471","Trade payables to associates","liability_current","l10n_se.account_tag_26","False","Leverantörsskulder till intresseföretag"
"a2472","2472","Trade payables to jointly controlled entities","liability_current","l10n_se.account_tag_26","False","Leverantörsskulder till gemensamt styrda företag"
"a2473","2473","Accounts payable to other enterprises in which there is an ownership interest.","liability_current","l10n_se.account_tag_27","False","Leverantörsskulder till övriga företag som det finns ett ägarintresse i"
"a2491","2491","Settlement of game organisers","liability_current","l10n_se.account_tag_21","False","Avräkning spelarrangörer"
"a2492","2492","Bills of exchange payable","liability_current","l10n_se.account_tag_25","False","Växelskulder"
"a2499","2499","Other other current liabilities","liability_current","l10n_se.account_tag_29","False","Andra övriga kortfristiga skulder"
"a2512","2512","Estimated income tax","liability_current","l10n_se.account_tag_28","False","Beräknad inkomstskatt"
"a2513","2513","Estimated property tax/property tax","liability_current","l10n_se.account_tag_28","False","Beräknad fastighetsskatt/fastighetsavgift"
"a2514","2514","Estimated special payroll tax on pension costs","liability_current","l10n_se.account_tag_28","False","Beräknad särskild löneskatt på pensionskostnader"
"a2515","2515","Estimated yield tax","liability_current","l10n_se.account_tag_28","False","Beräknad avkastningsskatt"
"a2517","2517","Estimated foreign tax","liability_current","l10n_se.account_tag_28","False","Beräknad utländsk skatt"
"a2518","2518","F-tax paid","liability_current","l10n_se.account_tag_28","False","Betald F-skatt"
"a2618","2618","Dormant output VAT, 25%","liability_current","l10n_se.account_tag_29","False","Vilande utgående moms, 25 %"
"a2628","2628","Suspended output VAT, 12 %","liability_current","l10n_se.account_tag_29","False","Vilande utgående moms, 12 %%"
"a2638","2638","Suspended output VAT, 6%","liability_current","l10n_se.account_tag_29","False","Vilande utgående moms, 6 %"
"a2660","2660","Excise duties","liability_current","l10n_se.account_tag_29","False","Punktskatter"
"a2661","2661","Advertising tax","liability_current","l10n_se.account_tag_29","False","Reklamskatt"
"a2669","2669","Other excise duties","liability_current","l10n_se.account_tag_29","False","Övriga punktskatter"
"a2731","2731","Settlement of statutory social contributions","liability_current","l10n_se.account_tag_29","False","Avräkning lagstadgade sociala avgifter"
"a2732","2732","Settlement of special payroll tax","liability_current","l10n_se.account_tag_29","False","Avräkning särskild löneskatt"
"a2750","2750","Attachment of wages, etc.","liability_current","l10n_se.account_tag_29","False","Utmätning i lön m.m."
"a2760","2760","Attachment of wages, etc.","liability_current","l10n_se.account_tag_29","False","Utmätning i lön m.m."
"a2761","2761","Settlement of holiday pay","liability_current","l10n_se.account_tag_29","False","Avräkning semesterlöner"
"a2762","2762","Holiday pay fund","liability_current","l10n_se.account_tag_29","False","Semesterlönekassa"
"a2791","2791","Staff interest account","liability_current","l10n_se.account_tag_29","False","Personalens intressekonto"
"a2792","2792","Salary savings","liability_current","l10n_se.account_tag_29","False","Lönsparande"
"a2793","2793","Group insurance premiums","liability_current","l10n_se.account_tag_29","False","Gruppförsäkringspremier"
"a2794","2794","Trade union dues","liability_current","l10n_se.account_tag_29","False","Fackföreningsavgifter"
"a2795","2795","Measurement and inspection fees","liability_current","l10n_se.account_tag_29","False","Mätnings- och granskningsarvoden"
"a2799","2799","Other payroll deductions","liability_current","l10n_se.account_tag_29","False","Övriga löneavdrag"
"a2810","2810","Settlement of factoring and mortgaged contractual receivables","liability_payable","l10n_se.account_tag_29","True","Avräkning för factoring och belånade kontraktsfordringar"
"a2811","2811","Settlement for factoring","liability_payable","l10n_se.account_tag_29","True","Avräkning för factoring"
"a2812","2812","Settlement for mortgaged contract claims","liability_payable","l10n_se.account_tag_29","True","Avräkning för belånade kontraktsfordringar"
"a2821","2821","Payroll liabilities","liability_payable","l10n_se.account_tag_29","True","Löneskulder"
"a2822","2822","Travel accounts","liability_payable","l10n_se.account_tag_29","True","Reseräkningar"
"a2823","2823","Bonuses, gratuities","liability_payable","l10n_se.account_tag_29","True","Tantiem, gratifikationer"
"a2829","2829","Other current liabilities to employees","liability_payable","l10n_se.account_tag_29","True","Övriga kortfristiga skulder till anställda"
"a2830","2830","Settlement on behalf of others","liability_payable","l10n_se.account_tag_29","True","Avräkning för annans räkning"
"a2841","2841","Current portion of long-term liabilities","liability_payable","l10n_se.account_tag_29","True","Kortfristig del av långfristiga skulder"
"a2849","2849","Other short-term loan liabilities","liability_payable","l10n_se.account_tag_29","True","Övriga kortfristiga låneskulder"
"a2850","2850","Settlement of taxes and duties (tax account)","liability_payable","l10n_se.account_tag_29","True","Avräkning för skatter och avgifter (skattekonto)"
"a2852","2852","Deferral of VAT, employer's contributions and employee taxes","liability_payable","l10n_se.account_tag_29","True","Anståndsbelopp för moms, arbetsgivaravgifter och personalskatt"
"a2860","2860","Current liabilities to group companies","liability_payable","l10n_se.account_tag_24","True","Kortfristiga skulder till koncernföretag"
"a2861","2861","Current liabilities to parent company","liability_payable","l10n_se.account_tag_24","True","Kortfristiga skulder till moderföretag"
"a2862","2862","Current liabilities to subsidiaries","liability_payable","l10n_se.account_tag_24","True","Kortfristiga skulder till dotterföretag"
"a2863","2863","Current liabilities to other group companies","liability_payable","l10n_se.account_tag_24","True","Kortfristiga skulder till andra koncernföretag"
"a2870","2870","Current liabilities to associates, jointly controlled entities and other entities in which there is an ownership interest","liability_payable","l10n_se.account_tag_26","True","Kortfristiga skulder till intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"a2871","2871","Current liabilities to associates","liability_payable","l10n_se.account_tag_26","True","Kortfristiga skulder till intresseföretag"
"a2872","2872","Current liabilities to jointly controlled entities","liability_payable","l10n_se.account_tag_26","True","Kortfristiga skulder till gemensamt styrda företag"
"a2873","2873","Current liabilities to other entities in which there is an ownership interest","liability_payable","l10n_se.account_tag_27","True","Kortfristiga skulder till övriga företag som det finns ett ägarintresse i"
"a2880","2880","Liability for grants received","liability_current","l10n_se.account_tag_29","False","Skuld erhållna bidrag"
"a2891","2891","Liabilities under recovery","liability_current","l10n_se.account_tag_29","False","Skulder under indrivning"
"a2892","2892","Internal repair fund/maintenance fund","liability_current","l10n_se.account_tag_29","False","Inre reparationsfond/underhållsfond"
"a2893","2893","Liabilities to related parties, current portion","liability_current","l10n_se.account_tag_29","False","Skulder till närstående personer, kortfristig del"
"a2895","2895","Derivatives (current liabilities)","liability_current","l10n_se.account_tag_29","False","Derivat (kortfristiga skulder)"
"a2897","2897","Deposits received, short-term","liability_current","l10n_se.account_tag_29","False","Mottagna depositioner, kortfristiga"
"a2898","2898","Withdrawn dividends","liability_current","l10n_se.account_tag_29","False","Outtagen vinstutdelning"
"a2899","2899","Other current liabilities","liability_current","l10n_se.account_tag_29","False","Övriga kortfristiga skulder"
"a2911","2911","Payroll liabilities","liability_current","l10n_se.account_tag_30","False","Löneskulder"
"a2912","2912","Retained earnings","liability_current","l10n_se.account_tag_30","False","Ackordsöverskott"
"a2919","2919","Other Accrued salaries and wages","liability_current","l10n_se.account_tag_30","False","Övriga Upplupna löner"
"a2930","2930","Accrued pension costs","liability_current","l10n_se.account_tag_30","False","Upplupna pensionskostnader"
"a2931","2931","Accrued pension payments","liability_current","l10n_se.account_tag_30","False","Upplupna pensionsutbetalningar"
"a2941","2941","Accrued accrued statutory social security contributions","liability_current","l10n_se.account_tag_30","False","Upplupna upplupna lagstadgade sociala avgifter"
"a2942","2942","Estimated accrued special payroll taxes","liability_current","l10n_se.account_tag_30","False","Beräknad upplupen särskild löneskatt"
"a2943","2943","Estimated accrued special payroll tax on pension costs, declaration item","liability_current","l10n_se.account_tag_30","False","Beräknad upplupen särskild löneskatt på pensionskostnader, deklarationspost"
"a2944","2944","Estimated accrued yield tax on pension costs","liability_current","l10n_se.account_tag_30","False","Beräknad upplupen avkastningsskatt på pensionskostnader"
"a2951","2951","Accrued contracted labour market insurances","liability_current","l10n_se.account_tag_30","False","Upplupna avtalade arbetsmarknadsförsäkringar"
"a2959","2959","Accrued contractual pension insurance contributions, declaration item","liability_current","l10n_se.account_tag_30","False","Upplupna avtalade pensionsförsäkringsavgifter, deklarationspost"
"a2971","2971","Prepaid rental income","liability_current","l10n_se.account_tag_30","False","Förutbetalda hyresintäkter"
"a2972","2972","Prepaid membership fees","liability_current","l10n_se.account_tag_30","False","Förutbetalda medlemsavgifter"
"a2979","2979","Other prepaid income","liability_current","l10n_se.account_tag_30","False","Övriga förutbetalda intäkter"
"a2980","2980","Accrued contractual costs","liability_current","l10n_se.account_tag_30","False","Upplupna avtalskostnader"
"a2991","2991","Estimated fee for financial statements","liability_current","l10n_se.account_tag_30","False","Beräknat arvode för bokslut"
"a2992","2992","Estimated fees for auditing","liability_current","l10n_se.account_tag_30","False","Beräknat arvode för revision"
"a2993","2993","Unspecified debt to suppliers","liability_current","l10n_se.account_tag_30","False","Ospecificerad skuld till leverantörer"
"a2998","2998","Other accrued expenses and deferred income","liability_current","l10n_se.account_tag_30","False","Övriga upplupna kostnader och förutbetalda intäkter"
"a3511","3511","Invoiced packaging","income","l10n_se.account_tag_31","False","Fakturerat emballage"
"a3518","3518","Packaging returned","income","l10n_se.account_tag_31","False","Returnerat emballage"
"a3560","3560","Costs invoiced to group companies","income","l10n_se.account_tag_31","False","Fakturerad kostnader till koncernföretag"
"a3561","3561","Costs invoiced to parent company","income","l10n_se.account_tag_31","False","Fakturerad kostnader till moderföretag"
"a3562","3562","Costs invoiced to subsidiaries","income","l10n_se.account_tag_31","False","Fakturerad kostnader till dotterföretag"
"a3563","3563","Costs invoiced to other group companies","income","l10n_se.account_tag_31","False","Fakturerad kostnader till andra koncernföretag"
"a3570","3570","Costs invoiced to associates, jointly controlled entities and other entities in which there is an ownership interest","income","l10n_se.account_tag_31","False","Fakturerad kostnader till intresseföretag, gemensamt styrda företag och övriga företag som det finns ägarintresse i"
"a3590","3590","Other invoiced expenses","income","l10n_se.account_tag_31","False","Övriga fakturerade kostnader"
"a3610","3610","Sale of materials","income","l10n_se.account_tag_31","False","Försäljning av material"
"a3611","3611","Sale of raw materials","income","l10n_se.account_tag_31","False","Försäljning av råmaterial"
"a3612","3612","Sale of scrap metal","income","l10n_se.account_tag_31","False","Försäljning av skrot"
"a3613","3613","Sales of consumables","income","l10n_se.account_tag_31","False","Försäljning av förbrukningsmaterial"
"a3619","3619","Sales of other materials","income","l10n_se.account_tag_31","False","Försäljning av övrigt material"
"a3620","3620","Temporary hiring of personnel","income","l10n_se.account_tag_31","False","Tillfällig uthyrning av personal"
"a3630","3630","Temporary use of transport equipment","income","l10n_se.account_tag_31","False","Tillfällig av transportmedel"
"a3670","3670","Income from securities","income","l10n_se.account_tag_31","False","Intäkter från värdepapper"
"a3671","3671","Sales of securities","income","l10n_se.account_tag_31","False","Försäljning av värdepapper"
"a3672","3672","Dividends from securities","income","l10n_se.account_tag_31","False","Utdelning från värdepapper"
"a3679","3679","other income from securities","income","l10n_se.account_tag_31","False","övriga intäkter från värdepapper"
"a3680","3680","Management fees","income","l10n_se.account_tag_31","False","Management fees"
"a3690","3690","Other ancillary income","income","l10n_se.account_tag_31","False","Övriga sidointäkter"
"a3700","3700","Income adjustments (group account)","income","l10n_se.account_tag_31","False","Intäktskorrigeringar (gruppkonto)"
"a3710","3710","Unearned income reductions","income","l10n_se.account_tag_31","False","Oförelade intäktsreduktioner"
"a3731","3731","Cash discounts granted","income","l10n_se.account_tag_31","False","Lämnade kassarabatter"
"a3732","3732","Volume discounts granted","income","l10n_se.account_tag_31","False","Lämnade mängdrabatter"
"a3750","3750","Excise duties","income","l10n_se.account_tag_31","False","Punktskatter"
"a3751","3751","Excise duties recognised as revenue (credit account)","income","l10n_se.account_tag_31","False","Intäktsförda punktskatter (kreditkonto)"
"a3752","3752","Excise duties payable (debit account)","income","l10n_se.account_tag_31","False","Skuldförda punktskatter (debetkonto)"
"a3790","3790","other revenue adjustments","income","l10n_se.account_tag_31","False","övriga intäktskorrigeringar"
"a3840","3840","Work capitalised (materials)","income","l10n_se.account_tag_31","False","Aktiverat arbete (material)"
"a3850","3850","Work capitalised (overheads)","income","l10n_se.account_tag_31","False","Aktiverat arbete (omkostnader)"
"a3870","3870","Work capitalised (personnel)","income","l10n_se.account_tag_31","False","Aktiverat arbete (personal)"
"a3910","3910","Rental and leasing income","income_other","l10n_se.account_tag_31","False","Hyres- och arrendeintäkter"
"a3911","3911","Rental income","income_other","l10n_se.account_tag_31","False","Hyresintäkter"
"a3912","3912","Rental income","income_other","l10n_se.account_tag_31","False","Arrendeintäkter"
"a3914","3914","Other rental income subject to VAT","income_other","l10n_se.account_tag_31","False","Övriga momspliktiga hyresintäkter"
"a3920","3920","Commission, licence and royalty income","income_other","l10n_se.account_tag_31","False","Provisionsintäkter, licensintäkter och royalties"
"a3921","3921","Commission income","income_other","l10n_se.account_tag_31","False","Provisionsintäkter"
"a3922","3922","Licence fees and royalties","income_other","l10n_se.account_tag_31","False","Licensintäkter och royalties"
"a3925","3925","Franchise income","income_other","l10n_se.account_tag_31","False","Franchiseintäkter"
"a3950","3950","Recovered, previously written-off trade receivables","income_other","l10n_se.account_tag_31","False","Återvunna, tidigare, avskrivna kundfordringar"
"a3971","3971","Gain on disposal of intangible fixed assets","income_other","l10n_se.account_tag_31","False","Vinst vid avyttring av immateriella anläggningstillgångar"
"a3972","3972","Gain on disposal of land and buildings","income_other","l10n_se.account_tag_31","False","Vinst vid avyttring av byggnader och mark"
"a3973","3973","Gain on disposal of machinery and equipment","income_other","l10n_se.account_tag_31","False","Vinst vid avyttring av maskiner och inventarier"
"a3981","3981","EU grants received","income_other","l10n_se.account_tag_31","False","Erhållna EU-bidrag"
"a3985","3985","Government grants received","income_other","l10n_se.account_tag_31","False","Erhållna statliga bidrag"
"a3987","3987","Municipal grants received","income_other","l10n_se.account_tag_31","False","Erhållna kommunala bidrag"
"a3988","3988","Grants and reimbursements received for","income_other","l10n_se.account_tag_31","False","Erhållna bidrag och ersättningar för"
"a3989","3989","Other grants received","income_other","l10n_se.account_tag_31","False","Övriga erhållna bidrag"
"a3990","3990","Other reimbursements and income","income_other","l10n_se.account_tag_31","False","Övriga ersättningar och intäkter"
"a3991","3991","Conflict compensation","income_other","l10n_se.account_tag_31","False","Konfliktersättning"
"a3992","3992","Damages received","income_other","l10n_se.account_tag_31","False","Erhållna skadestånd"
"a3993","3993","Donations and gifts received","income_other","l10n_se.account_tag_31","False","Erhållna donationer och gåvor"
"a3994","3994","Insurance claims","income_other","l10n_se.account_tag_31","False","Försäkringsersättningar"
"a3995","3995","Payments received on account of operating liabilities","income_other","l10n_se.account_tag_31","False","Erhållet ackord på skulder av rörelsekaraktär"
"a3996","3996","Advertising subsidies received","income_other","l10n_se.account_tag_31","False","Erhållna reklambidrag"
"a3997","3997","Compensation for sick leave","income_other","l10n_se.account_tag_31","False","Sjuklöneersättning"
"a3999","3999","Other operating income","income_other","l10n_se.account_tag_31","False","Övriga rörelseintäkter"
"a4416","4416","Goods purchased in Sweden, reverse charge, 12 per cent","expense_direct_cost","l10n_se.account_tag_32","False","Inköpta varor i Sverige, omvänd skattskyldighet, 12 %"
"a4417","4417","Purchased goods in Sweden, reverse charge, 6%","expense_direct_cost","l10n_se.account_tag_32","False","Inköpta varor i Sverige, omvänd skattskyldighet, 6 %"
"a4425","4425","Purchase of services in Sweden, reverse charge, 25%","expense_direct_cost","l10n_se.account_tag_32","False","Inköp tjänster i Sverige, omvänd skattskyldighet, 25 %"
"a4730","4730","Discounts received","expense_direct_cost","l10n_se.account_tag_32","False","Erhållna rabatter"
"a4731","4731","Cash discounts received","expense_direct_cost","l10n_se.account_tag_32","False","Erhållna kassarabatter"
"a4732","4732","Received quantity discounts (incl. bonus)","expense_direct_cost","l10n_se.account_tag_32","False","Erhållna mängdrabatter (inkl. bonus)"
"a4733","4733","Received activity aid","expense_direct_cost","l10n_se.account_tag_32","False","Erhållna aktivitetsstöd"
"a4790","4790","Other reduction of purchase prices","expense_direct_cost","l10n_se.account_tag_32","False","Övriga reduktion av inköpspriser"
"a4944","4944","Change in work in progress, materials and outlays","expense_direct_cost","l10n_se.account_tag_31","False","Förändring produkter i arbete, material och utlägg"
"a4945","4945","Change in work in progress, overheads","expense_direct_cost","l10n_se.account_tag_31","False","Förändring produkter i arbete, omkostnader"
"a4947","4947","Change in work in progress, personnel costs","expense_direct_cost","l10n_se.account_tag_31","False","Förändring produkter i arbete, personalkostnader"
"a4974","4974","Change in work in progress, materials and expenses","expense_direct_cost","l10n_se.account_tag_31","False","Förändring pågående arbete, material och utlägg"
"a4975","4975","Change in work in progress, overheads","expense_direct_cost","l10n_se.account_tag_31","False","Förändring pågående arbete, omkostnader"
"a4977","4977","Change in work in progress, personnel costs","expense_direct_cost","l10n_se.account_tag_31","False","Förändring pågående arbete, personalkostnader"
"a4980","4980","Change in stocks of securities","expense_direct_cost","l10n_se.account_tag_31","False","Förändring av lager av värdepapper"
"a4981","4981","Cost of securities sold","expense_direct_cost","l10n_se.account_tag_31","False","Sålda värdepappers anskaffningsvärde"
"a4987","4987","Impairment of securities","expense_direct_cost","l10n_se.account_tag_31","False","Nedskrivning av värdepapper"
"a4988","4988","Reversal of impairment of securities","expense_direct_cost","l10n_se.account_tag_31","False","Återföring av nedskrivning av värdepapper"
"a5000","5000","Premises costs (group account)","expense","l10n_se.account_tag_32","False","Lokalkostnader (gruppkonto)"
"a5011","5011","Rent of office space","expense","l10n_se.account_tag_32","False","Hyra för kontorslokaler"
"a5012","5012","Rent of garages","expense","l10n_se.account_tag_32","False","Hyra för garage"
"a5013","5013","Rent of warehouses","expense","l10n_se.account_tag_32","False","Hyra för lagerlokaler"
"a5050","5050","Premises equipment","expense","l10n_se.account_tag_32","False","Lokaltillbehör"
"a5061","5061","Cleaning services","expense","l10n_se.account_tag_32","False","Städning"
"a5062","5062","Refuse collection","expense","l10n_se.account_tag_32","False","Sophämtning"
"a5063","5063","Hire of rubbish containers","expense","l10n_se.account_tag_32","False","Hyra för sopcontainer"
"a5064","5064","Snow removal","expense","l10n_se.account_tag_32","False","Snöröjning"
"a5065","5065","Gardening services","expense","l10n_se.account_tag_32","False","Trädgårdsskötsel"
"a5090","5090","Other premises costs","expense","l10n_se.account_tag_32","False","Övriga lokalkostnader"
"a5098","5098","Other premises expenses, deductible","expense","l10n_se.account_tag_32","False","Övriga lokalkostnader, avdragsgilla"
"a5099","5099","Other premises costs, non-deductible","expense","l10n_se.account_tag_32","False","Övriga lokalkostnader, ej avdragsgilla"
"a5110","5110","Ground rent/leasehold charges","expense","l10n_se.account_tag_32","False","Tomträttsavgäld/arrende"
"a5131","5131","Heating","expense","l10n_se.account_tag_32","False","Uppvärmning"
"a5132","5132","Chimney sweeping","expense","l10n_se.account_tag_32","False","Sotning"
"a5161","5161","Cleaning","expense","l10n_se.account_tag_32","False","Städning"
"a5162","5162","Refuse collection","expense","l10n_se.account_tag_32","False","Sophämtning"
"a5163","5163","Rent for rubbish container","expense","l10n_se.account_tag_32","False","Hyra för sopcontainer"
"a5164","5164","Snow removal","expense","l10n_se.account_tag_32","False","Snöröjning"
"a5165","5165","Gardening","expense","l10n_se.account_tag_32","False","Trädgårdsskötsel"
"a5190","5190","Other property costs","expense","l10n_se.account_tag_32","False","Övriga fastighetskostnader"
"a5191","5191","Property tax/property fee","expense","l10n_se.account_tag_32","False","Fastighetsskatt/fastighetsavgift"
"a5192","5192","Property insurance premiums","expense","l10n_se.account_tag_32","False","Fastighetsförsäkringspremier"
"a5193","5193","Property maintenance and management","expense","l10n_se.account_tag_32","False","Fastighetsskötsel och förvaltning"
"a5198","5198","Other property costs, deductible","expense","l10n_se.account_tag_32","False","Övriga fastighetskostnader, avdragsgilla"
"a5199","5199","Other property costs, non-deductible","expense","l10n_se.account_tag_32","False","Övriga fastighetskostnader, ej avdragsgilla"
"a5210","5210","Renting of machinery and other technical equipment","expense","l10n_se.account_tag_32","False","Hyra av maskiner och andra tekniska anläggningar"
"a5211","5211","Short-term hire of machinery and equipment","expense","l10n_se.account_tag_32","False","Korttidshyra av maskiner och andra tekniska anläggningar"
"a5212","5212","Leasing of machinery and equipment","expense","l10n_se.account_tag_32","False","Leasing av maskiner och andra tekniska anläggningar"
"a5220","5220","Hire of computers","expense","l10n_se.account_tag_32","False","Hyra av datorer"
"a5221","5221","Short-term hire of equipment and tools","expense","l10n_se.account_tag_32","False","Korttidshyra av inventarier och verktyg"
"a5222","5222","Leasing of equipment and tools","expense","l10n_se.account_tag_32","False","Leasing av inventarier och verktyg"
"a5250","5250","Rental of computers","expense","l10n_se.account_tag_32","False","Hyra av datorer"
"a5251","5251","Short-term hire of computers","expense","l10n_se.account_tag_32","False","Korttidshyra av datorer"
"a5252","5252","Leasing of computers","expense","l10n_se.account_tag_32","False","Leasing av datorer"
"a5290","5290","Other rentals of fixed assets","expense","l10n_se.account_tag_32","False","Övriga hyreskostnader för anläggningstillgångar"
"a5310","5310","Electricity for operation","expense","l10n_se.account_tag_32","False","El för drift"
"a5320","5320","Gas","expense","l10n_se.account_tag_32","False","Gas"
"a5330","5330","Fuel oil","expense","l10n_se.account_tag_32","False","Eldningsolja"
"a5340","5340","Coal and coke","expense","l10n_se.account_tag_32","False","Stenkol och koks"
"a5350","5350","Peat, charcoal, wood and other wood fuels","expense","l10n_se.account_tag_32","False","Torv, träkol, ved och annat träbränsle"
"a5360","5360","Petrol, kerosene and motor fuel oil","expense","l10n_se.account_tag_32","False","Bensin, fotogen och motorbrännolja"
"a5370","5370","District heating, cooling and steam","expense","l10n_se.account_tag_32","False","Fjärrvärme, kyla och ånga"
"a5380","5380","Water","expense","l10n_se.account_tag_32","False","Vatten"
"a5390","5390","Other energy costs","expense","l10n_se.account_tag_32","False","Övriga energikostnader"
"a5411","5411","Consumable inventory with a useful life of more than one year","expense","l10n_se.account_tag_32","False","Förbrukningsinventarier med en livslängd på mer än ett år"
"a5412","5412","Consumable inventory with a useful life of one year or less","expense","l10n_se.account_tag_32","False","Förbrukningsinventarier led en livslängd på ett år eller mindre"
"a5430","5430","Transport equipment","expense","l10n_se.account_tag_32","False","Transportinventarier"
"a5440","5440","Consumable packaging","expense","l10n_se.account_tag_32","False","Förbrukningsemballage"
"a5480","5480","Work clothes and protective materials","expense","l10n_se.account_tag_32","False","Arbetskläder och skyddsmaterial"
"a5490","5490","Other consumable inventory and consumables","expense","l10n_se.account_tag_32","False","Övriga förbrukningsinventarier och förbrukningsmaterial"
"a5491","5491","Other consumables with a useful life of more than one year","expense","l10n_se.account_tag_32","False","Övriga förbrukningsinventarier med en livslängd på mer än ett år"
"a5492","5492","Other consumables with a useful life of one year or less","expense","l10n_se.account_tag_32","False","Övriga förbrukningsinventarier med en livslängd på ett år eller mindre"
"a5493","5493","Other consumables","expense","l10n_se.account_tag_32","False","Övrigt förbrukningsmaterial"
"a5510","5510","Repair and maintenance of machinery and equipment","expense","l10n_se.account_tag_32","False","Reparation och underhåll av maskiner och andra tekniska anläggningar"
"a5520","5520","Repair and maintenance of furniture, tools and computers etc.","expense","l10n_se.account_tag_32","False","Reparation och underhåll av inventarier, verktyg och datorer m.m"
"a5530","5530","Repair and maintenance of installations","expense","l10n_se.account_tag_32","False","Reparation och underhåll av installationer"
"a5550","5550","Repair and maintenance of consumable equipment","expense","l10n_se.account_tag_32","False","Reparation och underhåll av förbrukningsinventarier"
"a5580","5580","Maintenance and washing of work clothes","expense","l10n_se.account_tag_32","False","Underhåll och tvätt av arbetskläder"
"a5590","5590","Other repair and maintenance costs","expense","l10n_se.account_tag_32","False","Övriga kostnader för reparation och underhålla"
"a5610","5610","Passenger cars","expense","l10n_se.account_tag_32","False","Personbilar"
"a5616","5616","Congestion tax, deductible","expense","l10n_se.account_tag_32","False","Trängselskatt, avdragsgill"
"a5619","5619","Other passenger car costs","expense","l10n_se.account_tag_32","False","Övriga personbilskostnader"
"a5620","5620","Lorry costs","expense","l10n_se.account_tag_32","False","Lastbilskostnader"
"a5630","5630","Truck costs","expense","l10n_se.account_tag_32","False","Truckkostnader"
"a5640","5640","Working machinery costs","expense","l10n_se.account_tag_32","False","Kostnader för arbetsmaskiner"
"a5650","5650","Tractor costs","expense","l10n_se.account_tag_32","False","Traktorkostnader"
"a5660","5660","Motorbike, moped and scooter costs","expense","l10n_se.account_tag_32","False","Motorcykel-, moped-, och skoterkostnader"
"a5670","5670","Boat, aeroplane and helicopter costs","expense","l10n_se.account_tag_32","False","Båt-, flygplans- och helikopterkostnader"
"a5690","5690","Other transport equipment costs","expense","l10n_se.account_tag_32","False","Övriga kostnader för transportmedel"
"a5710","5710","Freight, transport and insurance for goods distribution","expense","l10n_se.account_tag_32","False","Frakter, transporter och försäkringar vid varudistribution"
"a5720","5720","Customs and freight forwarding costs, etc.","expense","l10n_se.account_tag_32","False","Tull- och speditionskostnader m.m."
"a5730","5730","Transport of labour","expense","l10n_se.account_tag_32","False","Arbetstransporter"
"a5790","5790","Other freight and transport costs","expense","l10n_se.account_tag_32","False","Övriga kostnader för frakter och transporter"
"a5830","5830","Host and accommodation","expense","l10n_se.account_tag_32","False","Host och logi"
"a5890","5890","other travelling expenses","expense","l10n_se.account_tag_32","False","övriga resekostnader"
"a5910","5910","Advertising","expense","l10n_se.account_tag_32","False","Annonsering"
"a5920","5920","Outdoor and traffic advertising","expense","l10n_se.account_tag_32","False","Utomhus- och trafikreklam"
"a5930","5930","Printed advertising material and direct mail","expense","l10n_se.account_tag_32","False","Reklamtrycksaker och direktreklam"
"a5940","5940","Exhibitions and fairs","expense","l10n_se.account_tag_32","False","Utställningar mässor"
"a5950","5950","In-store advertising and retailer advertising","expense","l10n_se.account_tag_32","False","Butiksreklam och återförsäljarreklam"
"a5960","5960","Product samples, promotional gifts, giveaways and competitions","expense","l10n_se.account_tag_32","False","Varuprover, reklamgåvor, presentreklam och tävlingar"
"a5970","5970","Film, radio, television and internet advertising","expense","l10n_se.account_tag_32","False","Film-, radio-, TV- och Internetreklam"
"a5980","5980","Public relations, institutional advertising and sponsorship","expense","l10n_se.account_tag_32","False","PR, institutionell reklam och sponsring"
"a5990","5990","Other advertising and public relations costs","expense","l10n_se.account_tag_32","False","Övriga kostnader for reklam och PR"
"a6000","6000","Other selling expenses","expense","l10n_se.account_tag_32","False","Övriga försäljningskostnader"
"a6010","6010","Catalogues, price lists, etc.","expense","l10n_se.account_tag_32","False","Kataloger, prislistor m. m."
"a6020","6020","Own trade journals","expense","l10n_se.account_tag_32","False","Egna facktidskrifter"
"a6030","6030","Special order costs","expense","l10n_se.account_tag_32","False","Speciella orderkostnader"
"a6040","6040","Credit card charges","expense","l10n_se.account_tag_32","False","Kontokortsavgifter"
"a6050","6050","Sales commissions","expense","l10n_se.account_tag_32","False","Försäljningsprovisioner"
"a6055","6055","Franchise costs, etc.","expense","l10n_se.account_tag_32","False","Franchisekostnader o.dyl."
"a6060","6060","Credit sales costs","expense","l10n_se.account_tag_32","False","Kreditförsäljningskostnader"
"a6061","6061","Credit information","expense","l10n_se.account_tag_32","False","Kreditupplysning"
"a6062","6062","Debt collection and KFM fees","expense","l10n_se.account_tag_32","False","Inkasso och KFM-avgifter"
"a6063","6063","Credit insurance premiums","expense","l10n_se.account_tag_32","False","Kreditförsäkringspremier"
"a6064","6064","Factoring fees","expense","l10n_se.account_tag_32","False","Factoringsavgifter"
"a6069","6069","Other credit sales costs","expense","l10n_se.account_tag_32","False","Övriga kreditförsäljningskostnader"
"a6070","6070","Representation","expense","l10n_se.account_tag_32","False","Representation"
"a6080","6080","Bank guarantees","expense","l10n_se.account_tag_32","False","Bankgarantier"
"a6110","6110","Office supplies","expense","l10n_se.account_tag_32","False","Kontorsmateriel"
"a6150","6150","Printed matter","expense","l10n_se.account_tag_32","False","Trycksaker"
"a6200","6200","Telephone and mail (group account)","expense","l10n_se.account_tag_32","False","Tele och post (gruppkonto)"
"a6211","6211","Fixed telephony","expense","l10n_se.account_tag_32","False","Fast telefoni"
"a6212","6212","Mobile phones","expense","l10n_se.account_tag_32","False","Mobiltelefon"
"a6213","6213","Mobile phone search","expense","l10n_se.account_tag_32","False","Mobilsökning"
"a6214","6214","Fax","expense","l10n_se.account_tag_32","False","Fax"
"a6215","6215","Telex","expense","l10n_se.account_tag_32","False","Telex"
"a6230","6230","Data communication","expense","l10n_se.account_tag_32","False","Datakommunikation"
"a6250","6250","Postal services","expense","l10n_se.account_tag_32","False","Postbefordran"
"a6300","6300","Business insurance and other risk costs (group account)","expense","l10n_se.account_tag_32","False","Företagsförsäkringar och övriga riskkostnader (gruppkonto)"
"a6320","6320","Deductibles in case of damage","expense","l10n_se.account_tag_32","False","Självrisker vid skada"
"a6330","6330","Losses in work in progress","expense","l10n_se.account_tag_32","False","Förluster i pågående arbeten"
"a6340","6340","Damages paid","expense","l10n_se.account_tag_32","False","Lämnade skadestånd"
"a6341","6341","Damages paid, deductible","expense","l10n_se.account_tag_32","False","Lämnade skadestånd, avdragsgilla"
"a6342","6342","Damages paid, non-deductible","expense","l10n_se.account_tag_32","False","Lämnade skadestånd, ej avdragsgilla"
"a6350","6350","Losses on trade receivables","expense","l10n_se.account_tag_32","False","Förluster på kundfordringar"
"a6351","6351","Recognised losses on trade receivables","expense","l10n_se.account_tag_32","False","Konstaterade förluster på kundfordringar"
"a6352","6352","Expected losses on trade receivables","expense","l10n_se.account_tag_32","False","Befarade förluster på kundfordringar"
"a6360","6360","Guarantee costs","expense","l10n_se.account_tag_32","False","Garantikostnader"
"a6361","6361","Change in guarantee provision","expense","l10n_se.account_tag_32","False","Förändring av garantiavsättning"
"a6362","6362","Actual guarantee costs","expense","l10n_se.account_tag_32","False","Faktiska garantikostnader"
"a6370","6370","Guarding and laming costs","expense","l10n_se.account_tag_32","False","Kostnader för bevakning och lam"
"a6380","6380","Losses on other short-term receivables","expense","l10n_se.account_tag_32","False","Förluster på övriga kortfristiga fordringar"
"a6400","6400","Management costs (group account)","expense","l10n_se.account_tag_32","False","Förvaltningskostnader (gruppkonto)"
"a6421","6421","Audit activities","expense","l10n_se.account_tag_32","False","Revision"
"a6422","6422","Audit activities other than auditing","expense","l10n_se.account_tag_32","False","Revisionsverksamhet utöver revision"
"a6423","6423","Tax consultancy - auditor","expense","l10n_se.account_tag_32","False","Skatterådgivning - revisor"
"a6424","6424","Other services - auditor","expense","l10n_se.account_tag_32","False","Övriga tjänster - revisor"
"a6430","6430","Management fees","expense","l10n_se.account_tag_32","False","Management fees"
"a6440","6440","Annual and interim reports","expense","l10n_se.account_tag_32","False","Årsredovisning och delårsrapporter"
"a6450","6450","General meeting of shareholders/annual or association meetings","expense","l10n_se.account_tag_32","False","Bolagsstämma/års- eller föreningsstämma"
"a6490","6490","Other management costs","expense","l10n_se.account_tag_32","False","Övriga förvaltningskostnader"
"a6500","6500","Other external services (group account)","expense","l10n_se.account_tag_32","False","Övriga externa tjänster (gruppkonto)"
"a6510","6510","Measurement costs","expense","l10n_se.account_tag_32","False","Mätningskostnader"
"a6520","6520","Drawing and copying costs","expense","l10n_se.account_tag_32","False","Ritnings-och kopieringskostnader"
"a6810","6810","Hired production staff","expense","l10n_se.account_tag_32","False","Inhyrd produktionspersonal"
"a6820","6820","Hired warehouse staff","expense","l10n_se.account_tag_32","False","Inhyrd lagerpersonal"
"a6830","6830","Hired transport personnel","expense","l10n_se.account_tag_32","False","Inhyrd transportpersonal"
"a6840","6840","Hired office and finance staff","expense","l10n_se.account_tag_32","False","Inhyrd kontors- och ekonomipersonal"
"a6850","6850","Hired IT staff","expense","l10n_se.account_tag_32","False","Inhyrd IT-personal"
"a6860","6860","Hired marketing and sales personnel","expense","l10n_se.account_tag_32","False","Inhyrd marknads- och försäljningspersonal"
"a6870","6870","Hired restaurant and shop staff","expense","l10n_se.account_tag_32","False","Inhyrd restaurang- och butikspersonal"
"a6880","6880","Hired business managers","expense","l10n_se.account_tag_32","False","Inhyrda företagsledare"
"a6890","6890","Other temporary staff","expense","l10n_se.account_tag_32","False","Övriga inhyrd personal"
"a6900","6900","Other external costs (group account)","expense","l10n_se.account_tag_32","False","Övriga externa kostnader (gruppkonto)"
"a6910","6910","Licence fees and royalties)","expense","l10n_se.account_tag_32","False","Licensavgifter och royalties)"
"a6920","6920","Cost of own patents","expense","l10n_se.account_tag_32","False","Kostnader för egna patent"
"a6930","6930","Cost of trade marks, etc.","expense","l10n_se.account_tag_32","False","Kostnader för varumärken m.m."
"a6940","6940","Inspection, testing and stamping fees","expense","l10n_se.account_tag_32","False","Kontroll-, provnings- och stämpelavgifter"
"a6950","6950","Supervision fees of authorities","expense","l10n_se.account_tag_32","False","Tillsynsavgifter myndigheter"
"a6981","6981","Association fees, deductible","expense","l10n_se.account_tag_32","False","Föreningsavgifter, avdragsgilla"
"a6982","6982","Association fees, non-deductible","expense","l10n_se.account_tag_32","False","Föreningsavgifter, ej avdragsgilla"
"a6990","6990","Other external costs","expense","l10n_se.account_tag_32","False","Övriga externa kostnader"
"a6993","6993","Grants and gifts received","expense","l10n_se.account_tag_32","False","Lämnade bidrag och gåvor"
"a6996","6996","Foreign income tax paid","expense","l10n_se.account_tag_32","False","Betald utländsk inkomstskatt"
"a6997","6997","Unpaid foreign income tax","expense","l10n_se.account_tag_32","False","Obetald utländsk inkomstskatt"
"a6998","6998","Foreign VAT","expense","l10n_se.account_tag_32","False","Utländsk moms"
"a6999","6999","Input VAT, mixed activities","expense","l10n_se.account_tag_32","False","Ingående moms, blandad verksamhet"
"a7000","7000","Wages and salaries of employees (group account)","expense","l10n_se.account_tag_32","False","Löner till kollektivanställda (gruppkonto)"
"a7010","7010","Wages and salaries to public sector employees","expense","l10n_se.account_tag_32","False","Löner till kollektivanställda"
"a7011","7011","Wages and salaries to public sector employees","expense","l10n_se.account_tag_32","False","Löner till kollektivanställda"
"a7012","7012","Profit shares to collective employees","expense","l10n_se.account_tag_32","False","Vinstandelar till kollektivanställda"
"a7013","7013","Salary growth support for collective employees 10.21 per cent","expense","l10n_se.account_tag_32","False","Lön växa-stöd kollektivanställda 10.21%"
"a7017","7017","Severance payments to collective employees","expense","l10n_se.account_tag_32","False","Avgångsvederlag till kollektivanställda"
"a7018","7018","Gross salary deductions, public sector employees","expense","l10n_se.account_tag_32","False","Bruttolöneavdrag, kollektivanställda"
"a7019","7019","Accrued wages and salaries and profit shares to collective employees","expense","l10n_se.account_tag_32","False","Upplupna löner och vinstandelar till kollektivanställda"
"a7030","7030","Wages and salaries to collective employees (expatriates)","expense","l10n_se.account_tag_32","False","Löner till kollektivanställda (utlandsanställda)"
"a7031","7031","Wages and salaries to collective employees (expatriates)","expense","l10n_se.account_tag_32","False","Löner till kollektivanställda (utlandsanställda)"
"a7032","7032","Profit shares to collective bargaining employees (expatriates)","expense","l10n_se.account_tag_32","False","Vinstandelar till kollektivanställda (utlandsanställda)"
"a7037","7037","Severance pay for collective workers (expatriates)","expense","l10n_se.account_tag_32","False","Avgångsvederlag till kollektivanställda (utlandsanställda)"
"a7038","7038","Gross salary deductions, collective bargaining employees (expatriates)","expense","l10n_se.account_tag_32","False","Bruttolöneavdrag, kollektivanställda (utlandsanställda)"
"a7039","7039","Accrued wages and salaries and profit shares to collective employees (expatriates)","expense","l10n_se.account_tag_32","False","Upplupna löner och vinstandelar till kollektivanställda (utlandsanställda)"
"a7080","7080","Wages and salaries to public sector employees for time not worked","expense","l10n_se.account_tag_32","False","Löner till kollektivanställda for ej arbetad tid"
"a7081","7081","Sick pay to collective employees","expense","l10n_se.account_tag_32","False","Sjuklöner till kollektivanställda"
"a7082","7082","Holiday pay for public sector employees","expense","l10n_se.account_tag_32","False","Semesterlöner till kollektivanställda"
"a7083","7083","Parental allowances for public sector employees","expense","l10n_se.account_tag_32","False","Föräldraersättning till kollektivanställda"
"a7089","7089","Other wages and salaries of employees in the public sector for non-working time","expense","l10n_se.account_tag_32","False","Övriga löner till kollektivanställda for ej arbetad tid"
"a7200","7200","Salaries of civil servants and company directors (group account)","expense","l10n_se.account_tag_32","False","Löner till tjänstemän och företagsledare (gruppkonto)"
"a7211","7211","Salaries to civil servants","expense","l10n_se.account_tag_32","False","Löner till tjänstemän"
"a7212","7212","Profit shares to civil servants","expense","l10n_se.account_tag_32","False","Vinstandelar till tjänstemän"
"a7213","7213","Salary growth support for officials 10.21","expense","l10n_se.account_tag_32","False","Lön växa-stöd tjänstemän 10.21%"
"a7217","7217","Severance pay to civil servants","expense","l10n_se.account_tag_32","False","Avgångsvederlag till tjänstemän"
"a7218","7218","Gross salary deductions to officials","expense","l10n_se.account_tag_32","False","Bruttolöneavdrag tjänstemän"
"a7219","7219","Accrued salaries and profit shares to civil servants","expense","l10n_se.account_tag_32","False","Upplupna löner och vinstandelar till tjänstemän"
"a7221","7221","Salaries to company directors","expense","l10n_se.account_tag_32","False","Löner till företagsledare"
"a7222","7222","Bonuses to company directors","expense","l10n_se.account_tag_32","False","Tantiem till företagsledare"
"a7227","7227","Severance pay to company directors","expense","l10n_se.account_tag_32","False","Avgångsvederlag till företagsledare"
"a7228","7228","Gross salary deductions, company directors","expense","l10n_se.account_tag_32","False","Bruttolöneavdrag, företagsledare"
"a7229","7229","Accrued wages and salaries and bonuses to company managers","expense","l10n_se.account_tag_32","False","Upplupna löner och tantiem till företagsledare"
"a7230","7230","Wages and salaries of officials and senior managers (expatriates)","expense","l10n_se.account_tag_32","False","Löner till tjänstemän och ftgsledare (utlandsanställda)"
"a7231","7231","Wages and salaries to officials and managers (expatriates)","expense","l10n_se.account_tag_32","False","Löner till tjänstemän och ftgsledare (utlandsanställda)"
"a7232","7232","Profit-sharing to officials and managers (expatriates)","expense","l10n_se.account_tag_32","False","Vinstandelar till tjänstemän och ftgsledare (utlandsanställda)"
"a7237","7237","Severance payments to officials (expatriates)","expense","l10n_se.account_tag_32","False","Avgångsvederlag till tjänstemän (utlandsanställda)"
"a7238","7238","Gross salary deductions, officials and trade union leaders (expatriates)","expense","l10n_se.account_tag_32","False","Bruttolöneavdrag, tjänstemän och ftgsledare (utlandsanställda)"
"a7239","7239","Accrued salaries and profit shares to officials and senior managers (expatriates)","expense","l10n_se.account_tag_32","False","Upplupna löner och vinstandelar till tjänstemän och ftgsledare (utlandsanställda)"
"a7280","7280","Salaries to officials and managers for time not worked","expense","l10n_se.account_tag_32","False","Löner till tjänstemän och företagsledare för ej arbetad tid"
"a7281","7281","Sick pay to officials","expense","l10n_se.account_tag_32","False","Sjuklöner till tjänstemän"
"a7282","7282","Sick pay for company directors","expense","l10n_se.account_tag_32","False","Sjuklöner till företagsledare"
"a7283","7283","Parental allowance for officials","expense","l10n_se.account_tag_32","False","Föräldraersättning till tjänstemän"
"a7284","7284","Parental allowances for company directors","expense","l10n_se.account_tag_32","False","Föräldraersättning till företagsledare"
"a7285","7285","Holiday pay for civil servants","expense","l10n_se.account_tag_32","False","Semesterlöner till tjänstemän"
"a7286","7286","Holiday pay for company directors","expense","l10n_se.account_tag_32","False","Semesterlöner till företagsledare"
"a7288","7288","Other salaries to civil servants for time not worked","expense","l10n_se.account_tag_32","False","Övriga löner till tjänsterumän för ej arbetad tid"
"a7289","7289","Other salaries to company directors for time not worked","expense","l10n_se.account_tag_32","False","Övriga löner till företagsledare för ej arbetad tid"
"a7291","7291","Change in holiday pay liability to salaried employees","expense","l10n_se.account_tag_32","False","Förändring av semesterlöneskuld till tjänstemän"
"a7292","7292","Change in holiday pay liability to company managers","expense","l10n_se.account_tag_32","False","Förändring av semesterlöneskuld till företagsledare"
"a7300","7300","Expenses and benefits (group account)","expense","l10n_se.account_tag_32","False","Kostnadsersättningar och förmåner (gruppkonto)"
"a7311","7311","Allowances for meetings, etc.","expense","l10n_se.account_tag_32","False","Ersättningar för sammanträden m.m."
"a7312","7312","Allowances for suggestions and inventions","expense","l10n_se.account_tag_32","False","Ersättningar för förslagsverksamhet och uppfinningar"
"a7313","7313","Reimbursements/allowances for housing costs","expense","l10n_se.account_tag_32","False","Ersättningar för/bidrag till bostadskostnader"
"a7314","7314","Reimbursement/reimbursement of meal costs","expense","l10n_se.account_tag_32","False","Ersättningar för/bidrag till måltidskostnader"
"a7315","7315","Reimbursements/allowances for travelling to and from the workplace","expense","l10n_se.account_tag_32","False","Ersättningar för/bidrag till resor till och från arbetsplatsen"
"a7316","7316","Reimbursement/reimbursement of work clothes","expense","l10n_se.account_tag_32","False","Ersättningar för/bidrag till arbetskläder"
"a7317","7317","Reimbursement/allowance for work materials and tools","expense","l10n_se.account_tag_32","False","Ersättningar för/bidrag till arbetsmaterial och arbetsverktyg"
"a7318","7318","Miscalculation money","expense","l10n_se.account_tag_32","False","Felräkningspengar"
"a7319","7319","Other additional cash allowances","expense","l10n_se.account_tag_32","False","Övriga kontanta extraersättningar"
"a7320","7320","Travel allowances for business trips","expense","l10n_se.account_tag_32","False","Traktamenten vid tjänsteresa"
"a7330","7330","Car allowances","expense","l10n_se.account_tag_32","False","Bilersättningar"
"a7333","7333","Compensation for congestion tax, tax-free","expense","l10n_se.account_tag_32","False","Ersättning för trängselskatt, skattefri"
"a7350","7350","Reimbursement for prescribed work clothes","expense","l10n_se.account_tag_32","False","Ersättningar för föreskrivna arbetskläder"
"a7370","7370","Hospitality allowances","expense","l10n_se.account_tag_32","False","Representationsersättningar"
"a7381","7381","Cost of free accommodation","expense","l10n_se.account_tag_32","False","Kostnader för fri bostad"
"a7382","7382","Cost of free or subsidised meals","expense","l10n_se.account_tag_32","False","Kostnader för fria eller subventionerade måltider"
"a7383","7383","Cost of free travel to and from the workplace","expense","l10n_se.account_tag_32","False","Kostnader för fria resor till och från arbetsplatsen"
"a7384","7384","Cost of free or subsidised work clothes","expense","l10n_se.account_tag_32","False","Kostnader för fria eller subventionerade arbetskläder"
"a7386","7386","Subsidised interest","expense","l10n_se.account_tag_32","False","Subventionerad ränta"
"a7387","7387","Cost of loaned computers","expense","l10n_se.account_tag_32","False","Kostnader för lånedatorer"
"a7388","7388","Employee compensation for benefits received","expense","l10n_se.account_tag_32","False","Anställdas ersättning för erhållna förmåner"
"a7389","7389","Other benefit costs","expense","l10n_se.account_tag_32","False","Övriga kostnader för förmåner"
"a7391","7391","Cost of wood tax benefit","expense","l10n_se.account_tag_32","False","Kostnad för trängelskatterförmån"
"a7392","7392","Cost of household services benefit","expense","l10n_se.account_tag_32","False","Kostnad för förmån av hushållsnära tjänster"
"a7400","7400","Pension costs (group account)","expense","l10n_se.account_tag_32","False","Pensionskostnader (gruppkonto)"
"a7411","7411","Premiums for collective pension schemes","expense","l10n_se.account_tag_32","False","Premier för kollektiva pensionsförsäkringar"
"a7412","7412","Premiums for individual pension insurance","expense","l10n_se.account_tag_32","False","Premier för individuella pensionsförsäkringar"
"a7418","7418","Refunds from insurance companies","expense","l10n_se.account_tag_32","False","Återbäring från försäkringsföretag"
"a7420","7420","Change in pension liability","expense","l10n_se.account_tag_32","False","Förändring av pensionsskuld"
"a7430","7430","Deduction of interest portion of pension cost","expense","l10n_se.account_tag_32","False","Avdrag för räntedel i pensionskostnad"
"a7440","7440","Change in pension fund","expense","l10n_se.account_tag_32","False","Förändring av pensionsstiftelse"
"a7441","7441","Provision to pension fund","expense","l10n_se.account_tag_32","False","Avsättning till pensionsstiftelse"
"a7448","7448","Compensation from pension fund","expense","l10n_se.account_tag_32","False","Gottgörelse från pensionsstiftelse"
"a7460","7460","Pension payments","expense","l10n_se.account_tag_32","False","Pensionsutbetalningar"
"a7461","7461","Pension payments to former public sector employees","expense","l10n_se.account_tag_32","False","Pensionsutbetalningar till f.d. kollektivanställda"
"a7462","7462","Pension payments to former civil servants","expense","l10n_se.account_tag_32","False","Pensionsutbetalningar till f.d. tjänstemän"
"a7463","7463","Pension payments to former company executives","expense","l10n_se.account_tag_32","False","Pensionsutbetalningar till f.d. företagsledare"
"a7470","7470","Management and credit insurance contribution","expense","l10n_se.account_tag_32","False","Förvaltnings-- och kreditförsäkringsavgift"
"a7500","7500","Social and other contributions according to law and agreements (group account)","expense","l10n_se.account_tag_32","False","Sociala och andra avgifter enlight lag och avtal (gruppkonto)"
"a7510","7510","Employer's contributions 31.42 per cent","expense","l10n_se.account_tag_32","False","Arbetsgivaravgifter 31.42%"
"a7515","7515","Employer's contributions on taxable expense reimbursements","expense","l10n_se.account_tag_32","False","Arbetsgivaravgifter på skattepliktiga kostnadsersättningar"
"a7516","7516","Employer's contributions on fees","expense","l10n_se.account_tag_32","False","Arbetsgivaravgifter på arvoden"
"a7518","7518","Employer's contributions on gross salary deductions","expense","l10n_se.account_tag_32","False","Arbetsgivaravgifter på bruttolöneavdrag"
"a7531","7531","Special payroll tax for certain insurance benefits, etc.","expense","l10n_se.account_tag_32","False","Särskild löneskatt för vissa försäkringsersättningar m.m."
"a7532","7532","Special payroll tax on pension costs, declaration item","expense","l10n_se.account_tag_32","False","Särskild löneskatt pensionskostnader, deklarationspost"
"a7533","7533","Special payroll tax for pension costs","expense","l10n_se.account_tag_32","False","Särskild löneskatt för pensionskostnader"
"a7551","7551","Income tax 15% insurance companies, etc. and allocated to pensions","expense","l10n_se.account_tag_32","False","Avkastningsskatt 15% försäkringsföretag m. lf. samt avsatt till pensioner"
"a7552","7552","Income tax 15% foreign pension insurance companies","expense","l10n_se.account_tag_32","False","Avkastningsskatt 15% utländska pensionsförsäkringar"
"a7553","7553","Withholding tax 30% foreign insurance companies etc.","expense","l10n_se.account_tag_32","False","Avkastningsskatt 30% utländska försäkringsföretag m. fl."
"a7554","7554","Yield tax 30% foreign endowment insurance","expense","l10n_se.account_tag_32","False","Avkastningsskatt 30% utländska kapitalförsäkringar"
"a7571","7571","Labour market insurance","expense","l10n_se.account_tag_32","False","Arbetsmarknadsförsäkringar"
"a7572","7572","Labour market insurance pension insurance premiums, declaration item","expense","l10n_se.account_tag_32","False","Arbetsmarknadsförsäkringar pensionsförsäkringspremier, deklarationspost"
"a7581","7581","Group life insurance premiums","expense","l10n_se.account_tag_32","False","Grupplivförsäkringspremier"
"a7582","7582","Group health insurance premiums","expense","l10n_se.account_tag_32","False","Gruppjukförsäkringspremier"
"a7583","7583","Group accident insurance premiums","expense","l10n_se.account_tag_32","False","Gruppolycksfallsförsäkringspremier"
"a7589","7589","Other group insurance premiums","expense","l10n_se.account_tag_32","False","Övriga gruppförsäkringspremier"
"a7620","7620","Medical and health care","expense","l10n_se.account_tag_32","False","Sjuk- och hälsovård"
"a7623","7623","Health insurance, non-deductible","expense","l10n_se.account_tag_32","False","Sjukvårdsförsäkring, ej avdragsgill"
"a7630","7630","Staff representation","expense","l10n_se.account_tag_32","False","Personalrepresentation"
"a7650","7650","Sick pay insurance","expense","l10n_se.account_tag_32","False","Sjuklöneförsäkring"
"a7670","7670","Change in capital of staff foundation","expense","l10n_se.account_tag_32","False","Förändring av personalstiftelsekapital"
"a7671","7671","Allocation to staff foundation","expense","l10n_se.account_tag_32","False","Avsättning till personalstiftelse"
"a7678","7678","Reimbursement from staff foundation of staff foundation capital","expense","l10n_se.account_tag_32","False","Gottgörelse från personalstiftelse av personalstiftelsekapital"
"a7690","7690","Other staff costs","expense","l10n_se.account_tag_32","False","Övriga personalkostnader"
"a7691","7691","Staff recruitment","expense","l10n_se.account_tag_32","False","Personalrekrytering"
"a7692","7692","Funeral assistance","expense","l10n_se.account_tag_32","False","Begravningshjälp"
"a7693","7693","Leisure activities","expense","l10n_se.account_tag_32","False","Fritidsverksamhet"
"a7699","7699","Other staff costs","expense","l10n_se.account_tag_32","False","Övriga personalkostnader"
"a7710","7710","Impairment of intangible fixed assets","expense","l10n_se.account_tag_32","False","Nedskrivningar av immateriella anläggningstillgångar"
"a7740","7740","Impairment of certain current assets","expense","l10n_se.account_tag_32","False","Nedskrivningar av vissa omsättningstillgångar"
"a7760","7760","Reversal of impairment of intangible fixed assets","expense","l10n_se.account_tag_32","False","Återföring av nedskrivningar av immateriella anläggningstillgångar"
"a7770","7770","Reversal of impairment losses on land and buildings","expense","l10n_se.account_tag_32","False","Återföring av nedskrivningar av byggnader och mark"
"a7780","7780","Reversal of impairment losses on machinery and equipment","expense","l10n_se.account_tag_32","False","Återföring av nedskrivningar av maskiner och inventarier"
"a7790","7790","Reversal of impairment losses on certain current assets","expense","l10n_se.account_tag_32","False","Återföring av nedskrivningar av vissa omsättningstillgångar"
"a7811","7811","Depreciation of capitalised expenditure","expense","l10n_se.account_tag_32","False","Avskrivningar på balanserade utgifter"
"a7812","7812","Amortisation of concessions, etc.","expense","l10n_se.account_tag_32","False","Avskrivningar på koncessioner m.m."
"a7813","7813","Amortisation of patents","expense","l10n_se.account_tag_32","False","Avskrivningar på patent"
"a7814","7814","Amortisation of licences","expense","l10n_se.account_tag_32","False","Avskrivningar på licenser"
"a7815","7815","Amortisation of trademarks","expense","l10n_se.account_tag_32","False","Avskrivningar på varumärken"
"a7816","7816","Amortisation of rental rights","expense","l10n_se.account_tag_32","False","Avskrivningar på hyresrätter"
"a7817","7817","Amortisation of goodwill","expense","l10n_se.account_tag_32","False","Avskrivningar på goodwill"
"a7819","7819","Amortisation of other intangible assets","expense","l10n_se.account_tag_32","False","Avskrivningar på övriga immateriella anläggningstillgångar"
"a7821","7821","Depreciation of buildings","expense","l10n_se.account_tag_32","False","Avskrivningar på byggnader"
"a7824","7824","Depreciation of land improvements","expense","l10n_se.account_tag_32","False","Avskrivningar på markanläggningar"
"a7829","7829","Depreciation of other buildings","expense","l10n_se.account_tag_32","False","Avskrivningar på övriga byggnader"
"a7831","7831","Depreciation of machinery and other technical equipment","expense","l10n_se.account_tag_32","False","Avskrivningar på maskiner och andra tekniska anläggningar"
"a7832","7832","Depreciation of equipment and tools","expense","l10n_se.account_tag_32","False","Avskrivningar på inventarier och verktyg"
"a7833","7833","Depreciation of installations","expense","l10n_se.account_tag_32","False","Avskrivningar på installationer"
"a7834","7834","Depreciation of cars and other transport equipment","expense","l10n_se.account_tag_32","False","Avskrivningar på bilar och nadra transportmedel"
"a7835","7835","Depreciation of computers","expense","l10n_se.account_tag_32","False","Avskrivningar på datorer"
"a7836","7836","Depreciation of leased assets","expense","l10n_se.account_tag_32","False","Avskrivningar på leasade tillgångar"
"a7839","7839","Depreciation of other machinery and equipment","expense","l10n_se.account_tag_32","False","Avskrivningar på övriga maskiner och inventarier"
"a7840","7840","Depreciation of improvements to property owned by others","expense","l10n_se.account_tag_32","False","Avskrivningar på förbättringsutgifter på annans fastighet"
"a7960","7960","Foreign exchange losses on operating receivables and liabilities","expense","l10n_se.account_tag_32","False","Valutakursförluster på fordringar och skulder av rörelsekaraktär"
"a7971","7971","Loss on the disposal of intangible fixed assets","expense","l10n_se.account_tag_32","False","Förlust vid avyttring av immateriella anläggningstillgångar"
"a7972","7972","Loss on disposal of land and buildings","expense","l10n_se.account_tag_32","False","Förlust vid avyttring av byggnader och mark"
"a7973","7973","Loss on disposal of machinery and equipment","expense","l10n_se.account_tag_32","False","Förlust vid avyttring av maskiner och inventarier"
"a8010","8010","Dividends on shares in group companies","income_other","l10n_se.account_tag_33","False","Utdelning på andelar i koncernföretag"
"a8012","8012","Dividends on shares in subsidiaries","income_other","l10n_se.account_tag_33","False","Utdelning på andelar i dotterföretag"
"a8016","8016","Share issue, group companies","income_other","l10n_se.account_tag_33","False","Emissionsinsats, koncernföretag"
"a8020","8020","Gain on sale of shares in group companies","income_other","l10n_se.account_tag_33","False","Resultat vid försäljning av andelar i koncernföretag"
"a8022","8022","Gain on sale of shares in subsidiaries","income_other","l10n_se.account_tag_33","False","Resultat vid försäljning av andelar i dotterföretag"
"a8030","8030","Income from partnerships (subsidiaries)","income_other","l10n_se.account_tag_33","False","Resultat från handelsbolag (dotterföretag)"
"a8070","8070","Impairment of shares in and long-term receivables from group companies","income_other","l10n_se.account_tag_33","False","Nedskrivningar av andelar i och långfristiga fordringar hos koncernföretag"
"a8072","8072","Impairment of shares in subsidiaries","income_other","l10n_se.account_tag_33","False","Nedskrivningar av andelar i dotterföretag"
"a8076","8076","Impairment of long-term receivables from parent companies","income_other","l10n_se.account_tag_33","False","Nedskrivningar av långfristiga fordringar hos moderföretag"
"a8077","8077","Impairment losses on long-term receivables from subsidiaries","income_other","l10n_se.account_tag_33","False","Nedskrivningar av långfristiga fordringar hos dotterföretag"
"a8080","8080","Reversals of impairment losses on investments in and long-term receivables from group companies","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av andelar i och långfristiga fordringar hos koncernföretag"
"a8082","8082","Reversals of impairment losses on shares in subsidiaries","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av andelar i dotterföretag"
"a8086","8086","Reversals of impairment losses on long-term receivables from parent companies","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av långfristiga fordringar hos moderföretag"
"a8087","8087","Reversals of impairment losses on long-term receivables from subsidiaries","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av långfristiga fordringar hos dotterföretag"
"a8110","8110","Dividends on shares in associates, jointly controlled entities and other enterprises in which there is a participating interest","income_other","l10n_se.account_tag_33","False","Utdelningar på andelar i intresseföretag, gemensamt styrda företag och Övriga företag som det finns ett ägarintresse i"
"a8111","8111","Dividends on shares in associates","income_other","l10n_se.account_tag_33","False","Utdelningar på andelar i intresseföretag"
"a8112","8112","Dividends on shares in jointly controlled entities","income_other","l10n_se.account_tag_33","False","Utdelningar på andelar i gemensamt styrda företag"
"a8113","8113","Dividends on shares in other enterprises in which there is an ownership interest","income_other","l10n_se.account_tag_33","False","Utdelningar på andelar i övriga företag som det finns ett ägarintresse i"
"a8116","8116","Equity contribution, associated enterprises","income_other","l10n_se.account_tag_33","False","Emissionsinsats, intresseföretag"
"a8117","8117","Equity contribution, jointly controlled entities","income_other","l10n_se.account_tag_33","False","Emissionsinsats, gemensamt styrda företag"
"a8118","8118","Share issue, other enterprises in which there is an ownership interest","income_other","l10n_se.account_tag_33","False","Emissionsinsats, övriga företag som det finns ett ägarintresse i"
"a8120","8120","Gain on sale of shares in associates, jointly controlled entities and other entities in which there is a participating interest","income_other","l10n_se.account_tag_33","False","Resultat vid försäljning av andelar i intresseFöretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"a8121","8121","Gain on sale of shares in associates","income_other","l10n_se.account_tag_33","False","Resultat vid försäljning av andelar i intresseföretag"
"a8122","8122","Gain on sale of shares in jointly controlled entities","income_other","l10n_se.account_tag_33","False","Resultat vid försäljning av andelar i gemensamt styrda företag"
"a8123","8123","Gains on the sale of shares in other enterprises in which there is a participating interest","income_other","l10n_se.account_tag_33","False","Resultat vid försäljning av andelar i övriga företag som det finns et ägarintresse i"
"a8130","8130","Share of profits of partnerships (associates, jointly controlled entities and other enterprises in which there is a participation)","income_other","l10n_se.account_tag_33","False","Resultatandelar från handelsbolag (intresseföretag, gemensamt styrda företag pcj övriga företag som det finns ett ägarintresse i)"
"a8131","8131","Profit shares from partnerships (associated companies)","income_other","l10n_se.account_tag_33","False","Resultatandelar från handelsbolag (intresseföretag)"
"a8132","8132","Share of profit or loss of partnerships (jointly controlled entities)","income_other","l10n_se.account_tag_33","False","Resultatandelar från handelsbolag (gemensamt styrda företag)"
"a8133","8133","Share of profit or loss of partnerships (other enterprises in which there is a participating interest)","income_other","l10n_se.account_tag_33","False","Resultatandelar från handelsbolag (övriga företag som det finns ett ägarintresse i)"
"a8170","8170","Impairment losses on participations and long-term receivables of associates, jointly controlled entities and other enterprises in which a participation is held","income_other","l10n_se.account_tag_33","False","Nedskrivningar av andelar i och långfristiga fordringar hos intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"a8171","8171","Impairment losses on investments in associates","income_other","l10n_se.account_tag_33","False","Nedskrivningar av andelar i intresseföretag"
"a8172","8172","Impairment of long-term receivables from associates","income_other","l10n_se.account_tag_33","False","Nedskrivningar av långfristiga fordringar hos intresseföretag"
"a8173","8173","Impairment of investments in jointly controlled entities","income_other","l10n_se.account_tag_33","False","Nedskrivningar av andelar i gemensamt styrda företag"
"a8174","8174","Impairment losses on long-term receivables from jointly controlled entities","income_other","l10n_se.account_tag_33","False","Nedskrivningar av långfristiga fordringar hos gemensamt styrda företag"
"a8176","8176","Impairment losses on participations in other enterprises in which there is an ownership interest","income_other","l10n_se.account_tag_33","False","Nedskrivningar av andelar i övriga företag som det finns ett ägarintresse i"
"a8177","8177","Impairment losses on long-term receivables from other enterprises in which a participating interest is held","income_other","l10n_se.account_tag_33","False","Nedskrivningar av långfristiga fordringar hos övriga företag som det finns et ägarintresse i"
"a8180","8180","Reversals of impairment losses on investments and long-term receivables from associates","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av andelar i och långfristiga fordringar hos intresseföretag"
"a8181","8181","Reversals of impairment losses on investments in associates","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av andelar i intresseföretag"
"a8182","8182","Reversals of impairment losses on long-term receivables from associates","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av långfristiga fordringar hos intresseföretag"
"a8183","8183","Reversals of impairment losses on shares in jointly controlled entities","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av andelar i gemensamt styrda företag"
"a8184","8184","Reversals of impairment losses on long-term receivables from jointly controlled entities","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av långfristiga fordringar hos gemensamt styrda företag"
"a8186","8186","Reversals of impairment losses on investments in other enterprises in which there is an ownership interest","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av andelar i övriga företag som det finns et ägarintresse i"
"a8187","8187","Reversals of impairment losses on long-term receivables from other enterprises in which there is a participating interest.","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av långfristiga fordringar hos övriga företag som det finns ett ägarintresse i"
"a8212","8212","Dividends, other enterprises","income_other","l10n_se.account_tag_33","False","Utdelningar, övriga företag"
"a8216","8216","Rights issues, other enterprises","income_other","l10n_se.account_tag_33","False","Insatsemissioner, övriga företag"
"a8221","8221","Gain on sale of shares in other enterprises","income_other","l10n_se.account_tag_33","False","Resultat vid försäljning av andelar i andra företag"
"a8222","8222","Gain on sale of long-term receivables from other enterprises","income_other","l10n_se.account_tag_33","False","Resultat vid försäljning av långfristiga fordringar hos andra företag"
"a8223","8223","Gain on sale of derivatives (long-term securities holdings)","income_other","l10n_se.account_tag_33","False","Resultat vid försäljning av derivat (långfristiga värdepappersinnehav)"
"a8230","8230","Exchange rate differences on long-term receivables","income_other","l10n_se.account_tag_33","False","Valutakursdifferenser på långfristiga fordringar"
"a8231","8231","Exchange rate gains on long-term receivables","income_other","l10n_se.account_tag_33","False","Valutakursvinster på långfristiga fordringar"
"a8236","8236","Foreign exchange losses on long-term receivables","income_other","l10n_se.account_tag_33","False","Valutakursförluster på långfristiga fordringar"
"a8240","8240","Share of profits of partnerships (other companies)","income_other","l10n_se.account_tag_33","False","Resultatandelar från handelsbolag (andra företag)"
"a8251","8251","Interest income on long-term receivables from group companies","income_other","l10n_se.account_tag_33","False","Ränteintäkter från långfristiga fordringar hos koncernföretag"
"a8252","8252","0.00Interest income from Other securities","income_other","l10n_se.account_tag_33","False","0.00Ränteintäkter från Övriga värdepapper"
"a8254","8254","Tax-free interest income, long-term assets","income_other","l10n_se.account_tag_33","False","Skattefria ränteintäkter, långfristiga tillgångar"
"a8255","8255","Income tax on capital investment","income_other","l10n_se.account_tag_33","False","Avkastningsskatt kapitalplacering"
"a8260","8260","Interest income on long-term receivables from group enterprises","income_other","l10n_se.account_tag_33","False","Ränteintäkter från långfristiga fordringar hos koncernföretag"
"a8261","8261","Interest income from long-term receivables from parent companies","income_other","l10n_se.account_tag_33","False","Ränteintäkter från långfristiga fordringar hos moderföretag"
"a8262","8262","Interest income on long-term receivables from subsidiaries","income_other","l10n_se.account_tag_33","False","Ränteintäkter från långfristiga fordringar hos dotterföretag"
"a8263","8263","Interest income on long-term receivables from other group companies","income_other","l10n_se.account_tag_33","False","Ränteintäkter från långfristiga fordringar hos andra koncernföretag"
"a8271","8271","Impairment of shares in other companies","income_other","l10n_se.account_tag_33","False","Nedskrivningar av andelar i andra företag"
"a8272","8272","Impairment of long-term receivables from other companies","income_other","l10n_se.account_tag_33","False","Nedskrivningar av långfristiga fordringar hos andra företag"
"a8273","8273","Impairment losses on other securities held by other enterprises","income_other","l10n_se.account_tag_33","False","Nedskrivningar av övriga värdepapper hos andra företag"
"a8280","8280","Reversals of participations in and long-term receivables from other corporations","income_other","l10n_se.account_tag_33","False","Återföringar av andelar i och långfristiga fordringar hos andra företag"
"a8281","8281","Reversals of impairment losses on shares in other corporations","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av av andelar i andra företag"
"a8282","8282","Reversals of impairment losses on long-term receivables from other enterprises","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av långfristiga fordringar hos andra företag"
"a8283","8283","Reversals of impairments on other securities held by other enterprises","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av övriga värdepapper i andra företag"
"a8311","8311","Interest income from banks","income_other","l10n_se.account_tag_33","False","Ränteintäkter från bank"
"a8312","8312","Interest income on short-term investments","income_other","l10n_se.account_tag_33","False","Ränteintäkter från kortfristiga placeringar"
"a8313","8313","Interest income on short-term receivables","income_other","l10n_se.account_tag_33","False","Ränteintäkter från kortfristiga fordringar"
"a8317","8317","Interest income for hidden interest rate compensation","income_other","l10n_se.account_tag_33","False","Ränteintäkter för dold räntekompensation"
"a8319","8319","Interest income on current assets","income_other","l10n_se.account_tag_33","False","Ränteintäkter från omsättningstillgångar"
"a8331","8331","Foreign exchange gains on short-term receivables and investments","income_other","l10n_se.account_tag_33","False","Valutakursvinster på kortfristiga fordringar och placeringar"
"a8336","8336","Foreign exchange losses on short-term receivables and investments","income_other","l10n_se.account_tag_33","False","Valutakursförluster på kortfristiga fordringar och placeringar"
"a8360","8360","Other interest income from group companies","income_other","l10n_se.account_tag_33","False","Övriga ränteintäkter från koncernföretag"
"a8361","8361","Other interest income from parent company","income_other","l10n_se.account_tag_33","False","Övriga ränteintäkter från moderföretag"
"a8362","8362","Other interest income from subsidiaries","income_other","l10n_se.account_tag_33","False","Övriga ränteintäkter från dotterföretag"
"a8363","8363","Other interest income from other group companies","income_other","l10n_se.account_tag_33","False","Övriga ränteintäkter från andra koncernföretag"
"a8370","8370","Impairment losses on short-term investments","income_other","l10n_se.account_tag_33","False","Nedskrivningar av kortfristiga placeringar"
"a8380","8380","Reversals of impairment losses on short-term investments","income_other","l10n_se.account_tag_33","False","Återföringar av nedskrivningar av kortfristiga placeringar"
"a8400","8400","Interest expense (group account)","income_other","l10n_se.account_tag_33","False","Räntekostnader (gruppkonto)"
"a8410","8410","Interest expense on long-term liabilities","income_other","l10n_se.account_tag_33","False","Räntekostnader för långfristiga skulder"
"a8411","8411","Interest expense on bonds, debentures and convertible loans","income_other","l10n_se.account_tag_33","False","Räntekostnader för obligations-, förlags- och konvertibla lån"
"a8412","8412","Interest element of the pension cost for the year","income_other","l10n_se.account_tag_33","False","Räntedel i årets pensionskostnad"
"a8413","8413","Interest expense on bank overdrafts","income_other","l10n_se.account_tag_33","False","Räntekostnader för checkräkningskredit"
"a8415","8415","Interest expense on other liabilities to credit institutions","income_other","l10n_se.account_tag_33","False","Räntekostnader för andra skulder till kreditinstitut"
"a8417","8417","Interest expense for hidden interest compensation, etc.","income_other","l10n_se.account_tag_33","False","Räntekostnader för dold räntekompensation m. m."
"a8418","8418","Interest expense for interest subsidies","income_other","l10n_se.account_tag_33","False","Räntekostnader för räntesubventioner"
"a8419","8419","Interest expense on long-term liabilities","income_other","l10n_se.account_tag_33","False","Räntekostnader för långfristiga skulder"
"a8421","8421","Interest payable to credit institutions","income_other","l10n_se.account_tag_33","False","Räntekostnader till kreditinstitut"
"a8424","8424","Interest expense on construction loans","income_other","l10n_se.account_tag_33","False","Räntekostnader byggnadskreditiv"
"a8429","8429","Other interest expense on short-term liabilities","income_other","l10n_se.account_tag_33","False","Övriga räntekostnader för kortfristiga skulder"
"a8431","8431","Foreign exchange gains on liabilities","income_other","l10n_se.account_tag_33","False","Valutakursvinster på skulder"
"a8436","8436","Exchange losses on liabilities","income_other","l10n_se.account_tag_33","False","Valutakursförluster på skulder"
"a8440","8440","Interest subsidies received","income_other","l10n_se.account_tag_33","False","Erhållna räntebidrag"
"a8460","8460","Interest expense to group companies","income_other","l10n_se.account_tag_33","False","Räntekostnader till koncernföretag"
"a8461","8461","Interest expense to parent company","income_other","l10n_se.account_tag_33","False","Räntekostnader till moderföretag"
"a8462","8462","Interest expense to subsidiaries","income_other","l10n_se.account_tag_33","False","Räntekostnader till dotterföretag"
"a8463","8463","Interest expense to other group companies","income_other","l10n_se.account_tag_33","False","Räntekostnader till andra koncernföretag"
"a8490","8490","Other liability-related items","income_other","l10n_se.account_tag_33","False","Övriga skuldrelaterade poster"
"a8491","8491","Settlement received on liabilities to credit institutions","income_other","l10n_se.account_tag_33","False","Erhållet ackord på skulder till kreditinstitut"
"a8810","8810","Change in tax allocation reserve","expense","l10n_se.account_tag_34","False","Förändring av periodiseringsfond"
"a8820","8820","Group contributions received","expense","l10n_se.account_tag_34","False","Mottagna koncernbidrag"
"a8830","8830","Group contributions paid","expense","l10n_se.account_tag_34","False","Lämnade koncernbidrag"
"a8840","8840","Indemnities paid","expense","l10n_se.account_tag_34","False","Lämnade gottgörelser"
"a8851","8851","Change in excess amortisation, intangible fixed assets","expense","l10n_se.account_tag_34","False","Förändring av överavskrivningar, immateriella anläggningstillgångar"
"a8852","8852","Change in excess depreciation, buildings and land improvements","expense","l10n_se.account_tag_34","False","Förändring av överskrivningar, byggnader och markanläggningar"
"a8853","8853","Change in excess depreciation, machinery and equipment","expense","l10n_se.account_tag_34","False","Förändring av överskrivningar, maskiner oh inventarier"
"a8860","8860","Change in compensation fund","expense","l10n_se.account_tag_34","False","Förändring av ersättningsfond"
"a8861","8861","Allocation to the replacement fund for equipment","expense","l10n_se.account_tag_34","False","Avsättning till ersättningsfond för inventarier"
"a8862","8862","Allocation to replacement fund for buildings and land improvements","expense","l10n_se.account_tag_34","False","Avsättning till ersättningsfond för byggnader och markanläggningar"
"a8864","8864","Allocation to the compensation fund for livestock in agriculture and reindeer husbandry","expense","l10n_se.account_tag_34","False","Avsättning till ersättningsfond för djurlager i jordbruk och renskötsel"
"a8865","8865","Utilisation of compensation fund for depreciation and amortisation","expense","l10n_se.account_tag_34","False","Ianspråktagande av ersättningsfond för avskrivningar"
"a8866","8866","Utilisation of compensation fund for other than depreciation and amortisation","expense","l10n_se.account_tag_34","False","Ianspråktagande av ersättningsfond fr annat än avskrivningar"
"a8869","8869","Transfer from compensation fund","expense","l10n_se.account_tag_34","False","Återföring fran ersättningsfond"
"a8890","8890","Other appropriations","expense","l10n_se.account_tag_34","False","Övriga bokslutsdispositioner"
"a8892","8892","Impairment losses on fixed assets of a consolidation nature","expense","l10n_se.account_tag_34","False","Nedskrivningar av konsolideringskaraktär av anläggningstillgångar"
"a8896","8896","Change in inventory reserve","expense","l10n_se.account_tag_34","False","Förändring av lagerreserv"
"a8899","8899","Other appropriations","expense","l10n_se.account_tag_34","False","Övriga bokslutsdispositioner"
"a8920","8920","Tax due to change in taxation","expense","l10n_se.account_tag_35","False","Skatt på grund av ändrad beskattning"
"a8930","8930","Taxes refunded","expense","l10n_se.account_tag_35","False","Restituerad skatt"
"a8980","8980","Other taxes","expense","l10n_se.account_tag_35","False","Övriga skatter"

```

## File: data\template\account.account-se_K3.csv

```csv
"id","code","name","account_type","tag_ids","reconcile","name@sv_SE"
"a1010","1010","Development expenditure","asset_non_current","l10n_se.account_tag_1","False","Utvecklingsutgifter"
"a1011","1011","Capitalised development expenditure","asset_non_current","l10n_se.account_tag_1","False","Balanserade utgifter för utveckling"
"a1012","1012","Capitalised expenditure on software","asset_non_current","l10n_se.account_tag_1","False","Balanserade utgifter för programvaror"
"a1018","1018","Accumulated impairment losses on capitalised expenditure","asset_non_current","l10n_se.account_tag_1","False","Ackumulerade nedskrivningar på balanserade utgifter"
"a1019","1019","Accumulated amortisation of capitalised expenditure","asset_non_current","l10n_se.account_tag_1","False","Ackumulerade avskrivningar på balanserade utgifter"
"a1081","1081","Projects in progress for intangible fixed assets","asset_non_current","l10n_se.account_tag_1","False","Pågående projekt för immateriella anläggningstillgångar"
"a1088","1088","Advances for intangible fixed assets","asset_non_current","l10n_se.account_tag_1","False","Förskott för immateriella anläggningstillgångar"
"a1260","1260","Leased assets","asset_non_current","l10n_se.account_tag_2","False","Leasade tillgångar"
"a1269","1269","Accumulated amortisation on leased assets","asset_non_current","l10n_se.account_tag_2","False","Ackumulerade avskrivningar på leasade tillgångar"
"a1370","1370","Deferred tax assets","asset_non_current","l10n_se.account_tag_3","False","Uppskjuten skattefordran"
"a2092","2092","Group contributions received/given","equity","l10n_se.account_tag_40","False","Mottagna/Lämnade koncernbidrag"
"a2096","2096","Fair value reserve","equity","l10n_se.account_tag_11","False","Fond för verkligt värde"
"a2240","2240","Provisions for deferred taxes","liability_non_current","l10n_se.account_tag_19","False","Avsättningar för uppskjutna skatter"
"a3940","3940","Unrealised negative/positive changes in value of hedging instruments","income_other","l10n_se.account_tag_31","False","Orealiserade negativa/positiva värdeförändringar på säkringsinstrument"
"a7940","7940","Unrealised positive/negative changes in value of hedging instruments","expense","l10n_se.account_tag_32","False","Orealiserade positiva/negativa värdeförändringar på säkringsinstrument"
"a8290","8290","Fair value measurement, fixed assets","income_other","l10n_se.account_tag_33","False","Värdering till verkligt värde, anläggningstillgångar"
"a8291","8291","Unrealised changes in value of fixed assets","income_other","l10n_se.account_tag_33","False","Orealiserade värdeförändringar på anläggningstillgångar"
"a8295","8295","Unrealised changes in value of derivative instruments","income_other","l10n_se.account_tag_33","False","Orealiserade värdeförändringar på derivatinstrument"
"a8320","8320","Fair value measurement of current assets","income_other","l10n_se.account_tag_33","False","Värdering till verkligt värde, omsättningstillgångar"
"a8321","8321","Unrealised changes in value of current assets","income_other","l10n_se.account_tag_33","False","Orealiserade värdeförändringar på omsättningstillgångar"
"a8325","8325","Interest income from current assets","income_other","l10n_se.account_tag_33","False","Ränteintäkter från omsättningstillgångar"
"a8450","8450","Unrealised changes in value of liabilities","income_other","l10n_se.account_tag_33","False","Orealiserade värdeförändringar på skulder"
"a8451","8451","Unrealised changes in value of liabilities","income_other","l10n_se.account_tag_33","False","Orealiserade värdeförändringar på skulder"
"a8455","8455","Unrealised changes in value of hedging instruments","income_other","l10n_se.account_tag_33","False","Orealiserade värdeförändringar på säkringsinstrument"
"a8480","8480","Capitalised interest expenses","income_other","l10n_se.account_tag_33","False","Aktiverade ränteutgifter"
"a8940","8940","Deferred tax","expense","l10n_se.account_tag_35","False","Uppskjuten skatt"

```

## File: data\template\account.fiscal.position-se.csv

```csv
"id","name","auto_apply","country_id","vat_required","sequence","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id"
"fp_sweden","Sverige","1","base.se","1","10","","","","",""
"fp_euro_b2c","Europaunionen (B2C)","1","","","11","base.europe","","","",""
"fp_euro_b2b","Europaunionen (B2B)","1","","1","12","base.europe","purchase_tax_25_services","purchase_services_tax_25_EC","",""
"","","","","","","","purchase_tax_25_goods","purchase_goods_tax_25_EC","",""
"","","","","","","","purchase_tax_12_services","purchase_services_tax_12_EC","",""
"","","","","","","","purchase_tax_12_goods","purchase_goods_tax_12_EC","",""
"","","","","","","","purchase_tax_6_services","purchase_services_tax_6_EC","",""
"","","","","","","","purchase_tax_6_goods","purchase_goods_tax_6_EC","",""
"","","","","","","","sale_tax_25_services","sale_tax_services_EC","",""
"","","","","","","","sale_tax_25_goods","sale_tax_goods_EC","",""
"","","","","","","","sale_tax_12_services","sale_tax_services_EC","",""
"","","","","","","","sale_tax_12_goods","sale_tax_goods_EC","",""
"","","","","","","","sale_tax_6_services","sale_tax_services_EC","",""
"","","","","","","","sale_tax_6_goods","sale_tax_goods_EC","",""
"","","","","","","","","","a3001","a3106"
"","","","","","","","","","a3002","a3106"
"","","","","","","","","","a3003","a3106"
"","","","","","","","","","a3004","a3106"
"","","","","","","","","","a3001","a3308"
"","","","","","","","","","a3002","a3308"
"","","","","","","","","","a3003","a3308"
"","","","","","","","","","a3004","a3308"
"fp_outside_euro","Utanför Europaunionen","1","","","13","","purchase_tax_25_services","purchase_services_tax_25_NEC","",""
"","","","","","","","purchase_tax_25_goods","purchase_goods_tax_25_NEC","",""
"","","","","","","","purchase_tax_12_services","purchase_services_tax_12_NEC","",""
"","","","","","","","purchase_tax_12_goods","purchase_goods_tax_12_NEC","",""
"","","","","","","","purchase_tax_6_services","purchase_services_tax_6_NEC","",""
"","","","","","","","purchase_tax_6_goods","purchase_goods_tax_6_NEC","",""
"","","","","","","","sale_tax_25_services","sale_tax_services_NEC","",""
"","","","","","","","sale_tax_25_goods","sale_tax_goods_NEC","",""
"","","","","","","","sale_tax_12_services","sale_tax_services_NEC","",""
"","","","","","","","sale_tax_12_goods","sale_tax_goods_NEC","",""
"","","","","","","","sale_tax_6_services","sale_tax_services_NEC","",""
"","","","","","","","sale_tax_6_goods","sale_tax_goods_NEC","",""
"","","","","","","","","","a3001","a3105"
"","","","","","","","","","a3002","a3105"
"","","","","","","","","","a3003","a3105"
"","","","","","","","","","a3004","a3105"
"","","","","","","","","","a3001","a3305"
"","","","","","","","","","a3002","a3305"
"","","","","","","","","","a3003","a3305"
"","","","","","","","","","a3004","a3305"

```

## File: data\template\account.group-se.csv

```csv
"id","code_prefix_start","name","name@sv_SE"
"se_group_1","1","Assets","Tillgångar"
"se_group_10","10","Intangible fixed assets","Immateriella anläggningstillgångar"
"se_group_101","101","Development expenditure","Utvecklingsutgifter"
"se_group_102","102","Concessions, etc.","Koncessioner m.m."
"se_group_103","103","Patents","Patent"
"se_group_104","104","Licences","Licenser"
"se_group_105","105","Trade marks","Varumärken"
"se_group_106","106","Tenancies, leasehold and similar rights","Hyresrätter, tomträtter och liknande"
"se_group_107","107","Goodwill","Goodwill"
"se_group_108","108","Advances for intangible fixed assets","Förskott för immateriella anläggningstillgångar"
"se_group_11","11","Buildings and land","Byggnader och mark"
"se_group_111","111","Buildings","Byggnader"
"se_group_112","112","Improvement costs on another one's property","Förbättringsutgifter på annans fastighet"
"se_group_113","113","Land","Mark"
"se_group_114","114","Land plots and undeveloped land","Tomter och obebyggda markområden"
"se_group_115","115","Land improvements","Markanläggningar"
"se_group_118","118","Construction in progress and advances for buildings and land","Pågående nyanläggningar och förskott för byggnader och mark"
"se_group_12","12","Machinery and equipment","Maskiner och inventarier"
"se_group_121","121","Machinery and other technical equipment","Maskiner och andra tekniska anläggningar"
"se_group_122","122","Equipment and tools","Inventarier och verktyg"
"se_group_123","123","Installations","Installationer"
"se_group_124","124","Vehicles and other means of transport","Bilar och andra transportmedel"
"se_group_125","125","Computers","Datorer"
"se_group_126","126","Leased assets","Leasade tillgångar"
"se_group_128","128","Construction in progress and advances for machinery and equipment","Pågående nyanläggningar och förskott för maskiner och inventarier"
"se_group_129","129","Other tangible fixed assets","Övriga materiella anläggningstillgångar"
"se_group_13","13","Financial fixed assets","Finansiella anläggningstillgångar"
"se_group_131","131","Shares in group companies","Andelar i koncernföretag"
"se_group_132","132","Long-term receivables from group companies","Långfristiga fordringar hos koncernföretag"
"se_group_133","133","Shares in associates, jointly controlled entities and other entities in which an ownership interest exists","Andelar i intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"se_group_134","134","Long-term receivables from associates, jointly controlled entities and other entities in which an ownership interest exists","Långfristiga fordringar hos intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"se_group_135","135","Shares and securities in other enterprises","Andelar och värdepapper i andra företag"
"se_group_136","136","Loans to partners or related parties according to ABL, long-term part","Lån till delägare eller närstående enligt ABL, långfristig del"
"se_group_137","137","Deferred tax assets","Uppskjuten skattefordran"
"se_group_138","138","Other long-term receivables","Andra långfristiga fordringar"
"se_group_14","14","Inventories, work in progress and work in progress","Lager, produkter i arbete och pågående arbeten"
"se_group_141","141","Stocks of raw materials","Lager av råvaror"
"se_group_142","142","Stocks of additives and consumables","Lager av tillsatsmaterial och förnödenheter"
"se_group_144","144","Work in progress","Produkter i arbete"
"se_group_145","145","Stocks of finished goods","Lager av färdiga varor"
"se_group_146","146","Stocks of merchandise","Lager av handelsvaror"
"se_group_147","147","Work in progress","Pågående arbeten"
"se_group_148","148","Advances for goods and services","Förskott för varor och tjänster"
"se_group_149","149","Other inventory assets","Övriga lagertillgångar"
"se_group_15","15","Trade receivables","Kundfordringar"
"se_group_151","151","Trade receivables","Kundfordringar"
"se_group_152","152","Bills of exchange receivable","Växelfordringar"
"se_group_153","153","Contract receivables","Kontraktsfordringar"
"se_group_155","155","Consignment receivables","Konsignationsfordringar"
"se_group_156","156","Accounts receivable from group companies","Kundfordringar hos koncernföretag"
"se_group_157","157","Accounts receivable from associates, jointly controlled entities and other entities in which there is an ownership interest.","Kundfordringar hos intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"se_group_158","158","Receivables for credit cards and vouchers","Fordringar för kontokort och kuponger"
"se_group_16","16","Other current receivables","Övriga kortfristiga fordringar"
"se_group_161","161","Short-term receivables from employees","Kortfristiga fordringar hos anställda"
"se_group_162","162","Accrued but uninvoiced revenue","Upparbetad men ej fakturerad intäkt"
"se_group_163","163","Offset for taxes and duties (tax account)","Avräkning för skatter och avgifter (skattekonto)"
"se_group_164","164","Taxes receivable","Skattefordringar"
"se_group_165","165","VAT receivable","Momsfordran"
"se_group_166","166","Current receivables from group companies","Kortfristiga fordringar hos koncernföretag"
"se_group_167","167","Current receivables from associates, jointly controlled entities and other entities in which there is an ownership interest.","Kortfristiga fordringar hos intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"se_group_168","168","Other current receivables","Andra kortfristiga fordringar"
"se_group_169","169","Receivables for subscribed but unpaid share capital","Fordringar för tecknat men ej inbetalt aktiekapital"
"se_group_17","17","Prepaid expenses and accrued income","Förutbetalda kostnader och upplupna intäkter"
"se_group_171","171","Prepaid rentals","Förutbetalda hyreskostnader"
"se_group_172","172","Prepaid leasing fees","Förutbetalda leasingavgifter"
"se_group_173","173","Prepaid insurance premiums","Förutbetalda försäkringspremier"
"se_group_174","174","Prepaid interest expenses","Förutbetalda räntekostnader"
"se_group_175","175","Accrued rental income","Upplupna hyresintäkter"
"se_group_176","176","Accrued interest income","Upplupna ränteintäkter"
"se_group_177","177","Assets of a cost nature","Tillgångar av kostnadsnatur"
"se_group_178","178","Accrued contractual income","Upplupna avtalsintäkter"
"se_group_179","179","Other prepaid expenses and accrued income","Övriga förutbetalda kostnader och upplupna intäkter"
"se_group_18","18","Short-term investments","Kortfristiga placeringar"
"se_group_181","181","Shares in listed companies","Andelar i börsnoterade företag"
"se_group_182","182","Bonds and notes","Obligationer"
"se_group_183","183","Convertible debt securities","Konvertibla skuldebrev"
"se_group_186","186","Shares in group companies, short-term","Andelar i koncernföretag, kortfristigt"
"se_group_188","188","Other short-term investments","Andra kortfristiga placeringar"
"se_group_189","189","Impairment of short-term investments","Nedskrivning av kortfristiga placeringar"
"se_group_19","19","Cash and bank balances","Kassa och bank"
"se_group_191","191","Cash in hand","Kassa"
"se_group_192","192","PlusGiro","PlusGiro"
"se_group_193","193","Company account/checking account/business account","Företagskonto/checkkonto/affärskonto"
"se_group_194","194","Other bank accounts","Övriga bankkonton"
"se_group_195","195","Bank certificates","Bankcertifikat"
"se_group_196","196","Group account parent company","Koncernkonto moderföretag"
"se_group_197","197","Special bank accounts","Särskilda bankkonton"
"se_group_198","198","Currency accounts","Valutakonton"
"se_group_199","199","Accounting funds","Redovisningsmedel"
"se_group_2","2","Equity and liabilities","Eget kapital och skulder"
"se_group_20","20","Equity capital","Eget kapital"
"se_group_201","201","Equity (partner 1/sole proprietorship)","Eget kapital (delägare 1/enskild firma)"
"se_group_202","202","Equity (partner 2)","Eget kapital (delägare 2)"
"se_group_203","203","Equity (partner 3)","Eget kapital (delägare 3)"
"se_group_204","204","Equity (partner 4)","Eget kapital (delägare 4)"
"se_group_205","205","Allocation to expansion fund","Avsättning till expansionsfond"
"se_group_206","206","Equity in non-profit organisations, foundations and registered religious communities","Eget kapital i ideella föreningar, stiftelser och registrerade trossamfund"
"se_group_207","207","Restricted funds","Ändamålsbestämda medel"
"se_group_208","208","Restricted equity","Bundet eget kapital"
"se_group_209","209","Unrestricted equity","Fritt eget kapital"
"se_group_21","21","Untaxed reserves","Obeskattade reserver"
"se_group_211","211","Accrual funds","Periodiseringsfonder"
"se_group_212","212","Tax allocation fund 2020","Periodiseringsfond 2020"
"se_group_213","213","Tax allocation reserve 2020 - No 2","Periodiseringsfond 2020 – nr 2"
"se_group_215","215","Accumulated excess depreciation","Ackumulerade överavskrivningar"
"se_group_216","216","Compensation fund","Ersättningsfond"
"se_group_219","219","Other untaxed reserves","Övriga obeskattade reserver"
"se_group_22","22","Provisions","Avsättningar"
"se_group_221","221","Provisions for pensions according to the Social Security Act","Avsättningar för pensioner enligt tryggandelagen"
"se_group_222","222","Provisions for guarantees","Avsättningar för garantier"
"se_group_223","223","Other provisions for pensions and similar obligations","Övriga avsättningar för pensioner och liknande förpliktelser"
"se_group_224","224","Provisions for deferred taxes","Avsättningar för uppskjutna skatter"
"se_group_225","225","Other provisions for taxes","Övriga avsättningar för skatter"
"se_group_229","229","Other provisions","Övriga avsättningar"
"se_group_23","23","Long-term liabilities","Långfristiga skulder"
"se_group_231","231","Bonds and debentures","Obligations- och förlagslån"
"se_group_232","232","Convertible loans and similar","Konvertibla lån och liknande"
"se_group_233","233","Bank overdrafts","Checkräkningskredit"
"se_group_234","234","Building loans","Byggnadskreditiv"
"se_group_235","235","Other long-term liabilities to credit institutions","Andra långfristiga skulder till kreditinstitut"
"se_group_236","236","Long-term liabilities to group companies","Långfristiga skulder till koncernföretag"
"se_group_237","237","Long-term liabilities to associates, jointly controlled entities and other entities in which there is an ownership interest","Långfristiga skulder till intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"se_group_239","239","Other long-term liabilities","Övriga långfristiga skulder"
"se_group_24","24","Current liabilities to credit institutions, customers and suppliers","Kortfristiga skulder till kreditinstitut, kunder och leverantörer"
"se_group_241","241","Other short-term loan liabilities to credit institutions","Andra kortfristiga låneskulder till kreditinstitut"
"se_group_242","242","Advances from customers","Förskott från kunder"
"se_group_243","243","Work in progress","Pågående arbeten"
"se_group_244","244","Trade payables","Leverantörsskulder"
"se_group_245","245","Invoiced but unrecognised revenue","Fakturerad men ej upparbetad intäkt"
"se_group_246","246","Trade payables to group companies","Leverantörsskulder till koncernföretag"
"se_group_247","247","Trade payables to associates, jointly controlled entities and other entities in which there is an ownership interest.","Leverantörsskulder till intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"se_group_248","248","Overdraft facility, short-term","Checkräkningskredit, kortfristig"
"se_group_249","249","Other current liabilities to credit institutions, customers and suppliers","Övriga kortfristiga skulder till kreditinstitut, kunder och leverantörer"
"se_group_25","25","Tax liabilities","Skatteskulder"
"se_group_251","251","Tax liabilities","Skatteskulder"
"se_group_26","26","VAT and excise duties","Moms och punktskatter"
"se_group_261","261","Outgoing VAT, 25 %","Utgående moms, 25 %"
"se_group_262","262","Output VAT, 12 %","Utgående moms, 12 %"
"se_group_263","263","Output VAT, 6 %","Utgående moms, 6 %"
"se_group_264","264","Input VAT","Ingående moms"
"se_group_265","265","Accounting account for VAT","Redovisningskonto för moms"
"se_group_266","266","Excise duties","Punktskatter"
"se_group_27","27","Staff taxes, contributions and payroll deductions","Personalens skatter, avgifter och löneavdrag"
"se_group_271","271","Employee taxes","Personalskatt"
"se_group_273","273","Statutory social security contributions and special payroll tax","Lagstadgade sociala avgifter och särskild löneskatt"
"se_group_274","274","Agreed social security contributions","Avtalade sociala avgifter"
"se_group_275","275","Attachment of wages, etc.","Utmätning i lön m.m."
"se_group_276","276","Holiday funds","Semestermedel"
"se_group_279","279","Other payroll deductions","Övriga löneavdrag"
"se_group_28","28","Other current liabilities","Övriga kortfristiga skulder"
"se_group_281","281","Settlement of factoring and mortgaged contract claims","Avräkning för factoring och belånade kontraktsfordringar"
"se_group_282","282","Current liabilities to employees","Kortfristiga skulder till anställda"
"se_group_283","283","Settlement on behalf of others","Avräkning för annans räkning"
"se_group_284","284","Short-term loan liabilities","Kortfristiga låneskulder"
"se_group_285","285","Settlement of taxes and duties (tax account)","Avräkning för skatter och avgifter (skattekonto)"
"se_group_286","286","Current liabilities to group companies","Kortfristiga skulder till koncernföretag"
"se_group_287","287","Current liabilities to associates, jointly controlled entities and other entities in which there is an ownership interest","Kortfristiga skulder till intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"se_group_288","288","Liability for grants received","Skuld erhållna bidrag"
"se_group_289","289","Other current liabilities","Övriga kortfristiga skulder"
"se_group_29","29","Accrued expenses and deferred income","Upplupna kostnader och förutbetalda intäkter"
"se_group_291","291","Accrued wages and salaries","Upplupna löner"
"se_group_292","292","Accrued holiday pay","Upplupna semesterlöner"
"se_group_293","293","Accrued pension costs","Upplupna pensionskostnader"
"se_group_294","294","Accrued statutory social security and other contributions","Upplupna lagstadgade sociala och andra avgifter"
"se_group_295","295","Accrued contractual social security contributions","Upplupna avtalade sociala avgifter"
"se_group_296","296","Accrued interest expenses","Upplupna räntekostnader"
"se_group_297","297","Deferred income","Förutbetalda intäkter"
"se_group_298","298","Accrued contractual costs","Upplupna avtalskostnader"
"se_group_299","299","Other accrued expenses and deferred income","Övriga upplupna kostnader och förutbetalda intäkter"
"se_group_3","3","Operating income/revenues","Rörelsens inkomster/intäkter"
"se_group_30","30","Main revenues","Huvudintäkter"
"se_group_300","300","Sales within Sweden","Försäljning inom Sverige"
"se_group_310","310","Sales of goods outside Sweden","Försäljning av varor utanför Sverige"
"se_group_320","320","Sales VMB and reverse charge VAT","Försäljning VMB och omvänd moms"
"se_group_321","321","Sales of positive VAT 25 per cent","Försäljning positiv VMB 25 %"
"se_group_323","323","Sales in the construction sector, reverse charge VAT","Försäljning inom byggsektorn, omvänd skattskyldighet moms"
"se_group_330","330","Sales of services outside Sweden","Försäljning av tjänster utanför Sverige"
"se_group_340","340","Sales, own consumption","Försäljning, egna uttag"
"se_group_35","35","Invoiced expenses","Fakturerade kostnader"
"se_group_350","350","Invoiced expenses (group account)","Fakturerade kostnader (gruppkonto)"
"se_group_351","351","Invoiced packaging","Fakturerat emballage"
"se_group_352","352","Invoiced freight","Fakturerade frakter"
"se_group_353","353","Invoiced customs and forwarding costs, etc.","Fakturerade tull- och speditionskostnader m.m."
"se_group_354","354","Invoicing charges","Faktureringsavgifter"
"se_group_355","355","Invoiced travelling expenses","Fakturerade resekostnader"
"se_group_356","356","Costs invoiced to group companies","Fakturerade kostnader till koncernföretag"
"se_group_357","357","Costs invoiced to associated companies, jointly controlled companies and other companies in which there is an ownership interest.","Fakturerade kostnader till intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"se_group_359","359","Other invoiced expenses","Övriga fakturerade kostnader"
"se_group_36","36","Ancillary operating income","Rörelsens sidointäkter"
"se_group_360","360","Ancillary operating income (group account)","Rörelsens sidointäkter (gruppkonto)"
"se_group_361","361","Sale of materials","Försäljning av material"
"se_group_362","362","Temporary rental of personnel","Tillfällig uthyrning av personal"
"se_group_363","363","Temporary rental of transport equipment","Tillfällig uthyrning av transportmedel"
"se_group_367","367","Income from securities","Intäkter från värdepapper"
"se_group_368","368","Management fees","Management fees"
"se_group_369","369","Other miscellaneous income","Övriga sidointäkter"
"se_group_37","37","Revenue adjustments","Intäktskorrigeringar"
"se_group_370","370","Revenue adjustments (group account)","Intäktskorrigeringar (gruppkonto)"
"se_group_371","371","Unallocated revenue reductions","Ofördelade intäktsreduktioner"
"se_group_373","373","Discounts granted","Lämnade rabatter"
"se_group_374","374","Öres and krona equalisation","Öres- och kronutjämning"
"se_group_375","375","Excise duties","Punktskatter"
"se_group_379","379","Other revenue adjustments","Övriga intäktskorrigeringar"
"se_group_38","38","Work capitalised on own account","Aktiverat arbete för egen räkning"
"se_group_380","380","Work capitalised for own account (group account)","Aktiverat arbete för egen räkning (gruppkonto)"
"se_group_384","384","Work capitalised (materials)","Aktiverat arbete (material)"
"se_group_385","385","Work capitalised (overheads)","Aktiverat arbete (omkostnader)"
"se_group_387","387","Work capitalised (personnel)","Aktiverat arbete (personal)"
"se_group_39","39","Other operating income","Övriga rörelseintäkter"
"se_group_390","390","Other operating income (group account)","Övriga rörelseintäkter (gruppkonto)"
"se_group_391","391","Rental and leasing income","Hyres- och arrendeintäkter"
"se_group_392","392","Commission, licence and royalty income","Provisionsintäkter, licensintäkter och royalties"
"se_group_394","394","Unrealised negative/positive changes in value of hedging instruments","Orealiserade negativa/positiva värdeförändringar på säkringsinstrument"
"se_group_395","395","Recovered, previously written off trade receivables","Återvunna, tidigare avskrivna kundfordringar"
"se_group_396","396","Exchange rate gains on receivables and liabilities of an operating nature","Valutakursvinster på fordringar och skulder av rörelsekaraktär"
"se_group_397","397","Gain on disposal of intangible and tangible fixed assets","Vinst vid avyttring av immateriella och materiella anläggningstillgångar"
"se_group_398","398","Government grants received","Erhållna offentliga bidrag"
"se_group_399","399","Other remuneration, grants and income","Övriga ersättningar, bidrag och intäkter"
"se_group_4","4","Expenditure/cost of goods, materials and certain purchased services","Utgifter/kostnader för varor, material och vissa köpta tjänster"
"se_group_40","40","Purchases of goods and materials","Inköp av varor och material"
"se_group_400","400","Purchases of goods from Sweden","Inköp av varor från Sverige"
"se_group_420","420","Goods sold VMB","Sålda varor VMB"
"se_group_421","421","Goods sold RMS 25 per cent","Sålda varor VMB 25 %"
"se_group_440","440","Purchases subject to VAT in Sweden","Momspliktiga inköp i Sverige"
"se_group_441","441","Goods purchased in Sweden, reverse charge, 25 % VAT.","Inköpta varor i Sverige, omvänd skattskyldighet, 25 % moms"
"se_group_442","442","Services purchased in Sweden, reverse charge, 25 % VAT","Inköpta tjänster i Sverige, omvänd skattskyldighet, 25 % moms"
"se_group_450","450","Other purchases subject to VAT","Övriga momspliktiga inköp"
"se_group_451","451","Purchase of goods from another EU country, 25%","Inköp av varor från annat EU-land, 25 %"
"se_group_453","453","Purchase of services from a country outside the EU, 25% VAT","Inköp av tjänster från ett land utanför EU, 25 % moms"
"se_group_454","454","Import of goods, 25% VAT","Import av varor, 25 % moms"
"se_group_46","46","Subcontracting, subcontracting","Legoarbeten, underentreprenader"
"se_group_460","460","Contract work and subcontracting (group account)","Legoarbeten och underentreprenader (gruppkonto)"
"se_group_47","47","Reduction of purchase prices","Reduktion av inköpspriser"
"se_group_470","470","Reduction of purchase prices (group account)","Reduktion av inköpspriser (gruppkonto)"
"se_group_473","473","Discounts received","Erhållna rabatter"
"se_group_479","479","Other purchase price reductions","Övriga reduktioner av inköpspriser"
"se_group_48","48","Free account group","Fri kontogrupp"
"se_group_49","49","Change in stocks, work in progress and work in progress","Förändring av lager, produkter i arbete och pågående arbeten"
"se_group_490","490","Change in stocks (group account)","Förändring av lager (gruppkonto)"
"se_group_491","491","Change in stocks of raw materials","Förändring av lager av råvaror"
"se_group_492","492","Change in stocks of additives and consumables","Förändring av lager av tillsatsmaterial och förnödenheter"
"se_group_494","494","Change in stocks of work in progress","Förändring av produkter i arbete"
"se_group_495","495","Change in stocks of finished goods","Förändring av lager av färdiga varor"
"se_group_496","496","Change in stocks of merchandise","Förändring av lager av handelsvaror"
"se_group_497","497","Change in work in progress, costs incurred","Förändring av pågående arbeten, nedlagda kostnader"
"se_group_498","498","Change in stocks of securities","Förändring av lager av värdepapper"
"se_group_5","5","Other external operating expenditure/expenses","Övriga externa rörelseutgifter/kostnader"
"se_group_50","50","Cost of premises","Lokalkostnader"
"se_group_500","500","Premises expenses (group account)","Lokalkostnader (gruppkonto)"
"se_group_501","501","Rent of premises","Lokalhyra"
"se_group_502","502","Electricity for lighting","El för belysning"
"se_group_503","503","Heating","Värme"
"se_group_504","504","Water and sewerage","Vatten och avlopp"
"se_group_505","505","Premises accessories","Lokaltillbehör"
"se_group_506","506","Cleaning and sanitation","Städning och renhållning"
"se_group_507","507","Repair and maintenance of premises","Reparation och underhåll av lokaler"
"se_group_509","509","Other premises costs","Övriga lokalkostnader"
"se_group_51","51","Real estate costs","Fastighetskostnader"
"se_group_510","510","Property costs (group account)","Fastighetskostnader (gruppkonto)"
"se_group_511","511","Ground rent/leasehold charges","Tomträttsavgäld/arrende"
"se_group_512","512","Electricity for lighting","El för belysning"
"se_group_513","513","Heating","Värme"
"se_group_514","514","Water and sewerage","Vatten och avlopp"
"se_group_516","516","Cleaning and sanitation","Städning och renhållning"
"se_group_517","517","Repair and maintenance of property","Reparation och underhåll av fastighet"
"se_group_519","519","Other property costs","Övriga fastighetskostnader"
"se_group_52","52","Rent of fixed assets","Hyra av anläggningstillgångar"
"se_group_520","520","Rental of fixed assets (group account)","Hyra av anläggningstillgångar (gruppkonto)"
"se_group_521","521","Rental of machinery and other technical equipment","Hyra av maskiner och andra tekniska anläggningar"
"se_group_522","522","Rent of equipment and tools","Hyra av inventarier och verktyg"
"se_group_525","525","Rental of computers","Hyra av datorer"
"se_group_529","529","Other rentals of fixed assets","Övriga hyreskostnader för anläggningstillgångar"
"se_group_53","53","Energy costs","Energikostnader"
"se_group_530","530","Energy costs (group account)","Energikostnader (gruppkonto)"
"se_group_531","531","Electricity for operation","El för drift"
"se_group_532","532","gas","Gas"
"se_group_533","533","Fuel oil","Eldningsolja"
"se_group_534","534","Coal and coke","Stenkol och koks"
"se_group_535","535","Peat, charcoal, wood and other wood fuels","Torv, träkol, ved och annat träbränsle"
"se_group_536","536","Petrol, kerosene and motor fuel oil","Bensin, fotogen och motorbrännolja"
"se_group_537","537","District heating, cooling and steam","Fjärrvärme, kyla och ånga"
"se_group_538","538","water","Vatten"
"se_group_539","539","Other energy costs","Övriga energikostnader"
"se_group_54","54","Consumable inventory and consumables","Förbrukningsinventarier och förbrukningsmaterial"
"se_group_540","540","Consumables and supplies (group account)","Förbrukningsinventarier och förbrukningsmaterial (gruppkonto)"
"se_group_541","541","Consumable inventory","Förbrukningsinventarier"
"se_group_542","542","Software","Programvaror"
"se_group_543","543","Transport equipment","Transportinventarier"
"se_group_544","544","Consumable packaging","Förbrukningsemballage"
"se_group_546","546","Consumables","Förbrukningsmaterial"
"se_group_548","548","Working clothes and protective materials","Arbetskläder och skyddsmaterial"
"se_group_549","549","Other consumables and supplies","Övriga förbrukningsinventarier och förbrukningsmaterial"
"se_group_55","55","Repair and maintenance","Reparation och underhåll"
"se_group_550","550","Repair and maintenance (group account)","Reparation och underhåll (gruppkonto)"
"se_group_551","551","Repair and maintenance of machinery and other technical equipment","Reparation och underhåll av maskiner och andra tekniska anläggningar"
"se_group_552","552","Repair and maintenance of equipment, tools and computers etc.","Reparation och underhåll av inventarier, verktyg och datorer m.m."
"se_group_553","553","Repair and maintenance of installations","Reparation och underhåll av installationer"
"se_group_555","555","Repair and maintenance of consumable equipment","Reparation och underhåll av förbrukningsinventarier"
"se_group_558","558","Maintenance and washing of work clothes","Underhåll och tvätt av arbetskläder"
"se_group_559","559","Other repair and maintenance costs","Övriga kostnader för reparation och underhåll"
"se_group_56","56","Cost of means of transport","Kostnader för transportmedel"
"se_group_560","560","Cost of means of transport (group account)","Kostnader för transportmedel (gruppkonto)"
"se_group_561","561","Passenger car costs","Personbilskostnader"
"se_group_562","562","lorry costs","Lastbilskostnader"
"se_group_563","563","Truck costs","Truckkostnader"
"se_group_564","564","Costs of working machines","Kostnader för arbetsmaskiner"
"se_group_565","565","Tractor costs","Traktorkostnader"
"se_group_566","566","Motorbike, moped and scooter costs","Motorcykel-, moped- och skoterkostnader"
"se_group_567","567","Boat, aeroplane and helicopter costs","Båt-, flygplans- och helikopterkostnader"
"se_group_569","569","Other means of transport costs","Övriga kostnader för transportmedel"
"se_group_57","57","Freight and transport","Frakter och transporter"
"se_group_570","570","Freight and transport (group account)","Frakter och transporter (gruppkonto)"
"se_group_571","571","Freight, transport and insurance in the distribution of goods","Frakter, transporter och försäkringar vid varudistribution"
"se_group_572","572","Customs and forwarding costs, etc.","Tull- och speditionskostnader m.m."
"se_group_573","573","Transport of labour","Arbetstransporter"
"se_group_579","579","Other freight and transport costs","Övriga kostnader för frakter och transporter"
"se_group_58","58","Travelling expenses","Resekostnader"
"se_group_580","580","Travelling expenses (group account)","Resekostnader (gruppkonto)"
"se_group_581","581","tickets","Biljetter"
"se_group_582","582","Car hire costs","Hyrbilskostnader"
"se_group_583","583","Food and accommodation","Kost och logi"
"se_group_589","589","Other travelling expenses","Övriga resekostnader"
"se_group_59","59","Advertising and public relations","Reklam och PR"
"se_group_590","590","Advertising and public relations (group account)","Reklam och PR (gruppkonto)"
"se_group_591","591","Advertising","Annonsering"
"se_group_592","592","Outdoor and traffic advertising","Utomhus- och trafikreklam"
"se_group_593","593","Printed advertising material and direct mail","Reklamtrycksaker och direktreklam"
"se_group_594","594","Exhibitions and fairs","Utställningar och mässor"
"se_group_595","595","In-store advertising and retailer advertising","Butiksreklam och återförsäljarreklam"
"se_group_596","596","Product samples, promotional gifts, giveaways and competitions","Varuprover, reklamgåvor, presentreklam och tävlingar"
"se_group_597","597","Film, radio, television and internet advertising","Film-, radio-, TV- och Internetreklam"
"se_group_598","598","Public relations, institutional advertising and sponsorship","PR, institutionell reklam och sponsring"
"se_group_599","599","Other advertising and public relations costs","Övriga kostnader för reklam och PR"
"se_group_6","6","Other external operating expenditure/costs","Övriga externa rörelseutgifter/kostnader"
"se_group_60","60","Other selling expenses","Övriga försäljningskostnader"
"se_group_600","600","Other selling expenses (group account)","Övriga försäljningskostnader (gruppkonto)"
"se_group_601","601","Catalogues, price lists, etc.","Kataloger, prislistor m.m."
"se_group_602","602","Own trade journals","Egna facktidskrifter"
"se_group_603","603","Special order costs","Speciella orderkostnader"
"se_group_604","604","Credit card charges","Kontokortsavgifter"
"se_group_605","605","Sales commissions","Försäljningsprovisioner"
"se_group_606","606","Credit sales costs","Kreditförsäljningskostnader"
"se_group_607","607","Representation","Representation"
"se_group_608","608","Bank guarantees","Bankgarantier"
"se_group_609","609","Other selling expenses","Övriga försäljningskostnader"
"se_group_61","61","Office supplies and printed matter","Kontorsmateriel och trycksaker"
"se_group_610","610","Office supplies and printed matter (group account)","Kontorsmateriel och trycksaker (gruppkonto)"
"se_group_611","611","Office supplies","Kontorsmateriel"
"se_group_615","615","Printed matter","Trycksaker"
"se_group_62","62","Telephone and mail","Tele och post"
"se_group_620","620","Telecommunications and post (group account)","Tele och post (gruppkonto)"
"se_group_621","621","Telecommunications","Telekommunikation"
"se_group_623","623","Data communications","Datakommunikation"
"se_group_625","625","Postal services","Postbefordran"
"se_group_63","63","Business insurance and other risk costs","Företagsförsäkringar och övriga riskkostnader"
"se_group_630","630","Business insurance and other risk charges (group account)","Företagsförsäkringar och övriga riskkostnader (gruppkonto)"
"se_group_631","631","Business insurance","Företagsförsäkringar"
"se_group_632","632","Deductibles in case of damage","Självrisker vid skada"
"se_group_633","633","Losses in work in progress","Förluster i pågående arbeten"
"se_group_634","634","Claims paid","Lämnade skadestånd"
"se_group_635","635","Losses on trade receivables","Förluster på kundfordringar"
"se_group_636","636","Guarantee costs","Garantikostnader"
"se_group_637","637","Security and alarm costs","Kostnader för bevakning och larm"
"se_group_638","638","Losses on other short-term receivables","Förluster på övriga kortfristiga fordringar"
"se_group_639","639","Other risk costs","Övriga riskkostnader"
"se_group_64","64","Management expenses","Förvaltningskostnader"
"se_group_640","640","Management expenses (group account)","Förvaltningskostnader (gruppkonto)"
"se_group_641","641","Non-salary directors' fees","Styrelsearvoden som inte är lön"
"se_group_642","642","Remuneration to the auditor","Ersättningar till revisor"
"se_group_643","643","Management fees","Management fees"
"se_group_644","644","Annual and interim reports","Årsredovisning och delårsrapporter"
"se_group_645","645","General meeting of shareholders/annual or association meetings","Bolagsstämma/års- ellerföreningsstämma"
"se_group_649","649","Other management costs","Övriga förvaltningskostnader"
"se_group_65","65","Other external services","Övriga externa tjänster"
"se_group_650","650","Other external services (group account)","Övriga externa tjänster (gruppkonto)"
"se_group_651","651","Measurement costs","Mätningskostnader"
"se_group_652","652","Drawing and copying costs","Ritnings- och kopieringskostnader"
"se_group_653","653","Accounting services","Redovisningstjänster"
"se_group_654","654","IT services","IT-tjänster"
"se_group_655","655","Consultancy fees","Konsultarvoden"
"se_group_656","656","Service fees to professional organisations","Serviceavgifter till branschorganisationer"
"se_group_657","657","Bank charges","Bankkostnader"
"se_group_658","658","Legal and court costs","Advokat- och rättegångskostnader"
"se_group_659","659","Other external services","Övriga externa tjänster"
"se_group_66","66","Free account group","Fri kontogrupp"
"se_group_67","67","Free account group","Fri kontogrupp"
"se_group_68","68","Temporary staff","Inhyrd personal"
"se_group_680","680","Temporary staff (group account)","Inhyrd personal (gruppkonto)"
"se_group_681","681","Hired production staff","Inhyrd produktionspersonal"
"se_group_682","682","Hired warehouse staff","Inhyrd lagerpersonal"
"se_group_683","683","Hired transport personnel","Inhyrd transportpersonal"
"se_group_684","684","Hired office and finance staff","Inhyrd kontors- och ekonomipersonal"
"se_group_685","685","Hired IT staff","Inhyrd IT-personal"
"se_group_686","686","Hired marketing and sales personnel","Inhyrd marknads- och försäljningspersonal"
"se_group_687","687","Hired restaurant and shop staff","Inhyrd restaurang- och butikspersonal"
"se_group_688","688","Temporary business managers","Inhyrda företagsledare"
"se_group_689","689","Other temporary staff","Övrig inhyrd personal"
"se_group_69","69","Other external costs","Övriga externa kostnader"
"se_group_690","690","Other external costs (group account)","Övriga externa kostnader (gruppkonto)"
"se_group_691","691","Licence fees and royalties","Licensavgifter och royalties"
"se_group_692","692","Cost of own patents","Kostnader för egna patent"
"se_group_693","693","Cost of trade marks, etc.","Kostnader för varumärken m.m."
"se_group_694","694","Inspection, testing and stamping fees","Kontroll-, provnings- och stämpelavgifter"
"se_group_695","695","Regulatory fees of authorities","Tillsynsavgifter myndigheter"
"se_group_697","697","Newspapers, periodicals and specialised literature","Tidningar, tidskrifter och facklitteratur"
"se_group_698","698","Association fees","Föreningsavgifter"
"se_group_699","699","Other external costs","Övriga externa kostnader"
"se_group_7","7","Expenditure/expenses for personnel, depreciation, etc.","Utgifter/kostnader för personal, avskrivningar m.m."
"se_group_70","70","Wages and salaries to collective employees","Löner till kollektivanställda"
"se_group_700","700","Wages and salaries of collective employees (group account)","Löner till kollektivanställda (gruppkonto)"
"se_group_701","701","Wages and salaries of public sector employees","Löner till kollektivanställda"
"se_group_703","703","Wages and salaries of collective employees (expatriates)","Löner till kollektivanställda (utlandsanställda)"
"se_group_708","708","Wages and salaries to collective employees for time not worked","Löner till kollektivanställda för ej arbetad tid"
"se_group_709","709","Change in holiday pay liability","Förändring av semesterlöneskuld"
"se_group_71","71","Free account group","Fri kontogrupp"
"se_group_72","72","Wages and salaries of officials and managers","Löner till tjänstemän och företagsledare"
"se_group_720","720","Wages and salaries of officials and managers (group account)","Löner till tjänstemän och företagsledare (gruppkonto)"
"se_group_721","721","Salaries to civil servants","Löner till tjänstemän"
"se_group_722","722","Salaries to company directors","Löner till företagsledare"
"se_group_723","723","Salaries of officials and company directors (expatriates)","Löner till tjänstemän och ftgsledare (utlandsanställda)"
"se_group_724","724","Directors' fees","Styrelsearvoden"
"se_group_728","728","Salaries of officials and managers for non-working time","Löner till tjänstemän och företagsledare för ej arbetad tid"
"se_group_729","729","Change in holiday pay liability","Förändring av semesterlöneskuld"
"se_group_73","73","Expense reimbursements and benefits","Kostnadsersättningar och förmåner"
"se_group_730","730","Allowances and benefits (group account)","Kostnadsersättningar och förmåner (gruppkonto)"
"se_group_731","731","Additional cash allowances","Kontanta extraersättningar"
"se_group_732","732","Mission allowances","Traktamenten vid tjänsteresa"
"se_group_733","733","Car allowances","Bilersättningar"
"se_group_735","735","Reimbursement of prescribed work clothes","Ersättningar för föreskrivna arbetskläder"
"se_group_737","737","Entertainment allowances","Representationsersättningar"
"se_group_738","738","Cost of employee benefits","Kostnader för förmåner till anställda"
"se_group_739","739","Other reimbursements and benefits","Övriga kostnadsersättningar och förmåner"
"se_group_74","74","Pension costs","Pensionskostnader"
"se_group_740","740","Pension costs (group account)","Pensionskostnader (gruppkonto)"
"se_group_741","741","Pension insurance premiums","Pensionsförsäkringspremier"
"se_group_742","742","Change in pension liability","Förändring av pensionsskuld"
"se_group_743","743","Deduction of interest portion of pension cost","Avdrag för räntedel i pensionskostnad"
"se_group_744","744","Change in pension fund capital","Förändring av pensionsstiftelsekapital"
"se_group_746","746","Pension payments","Pensionsutbetalningar"
"se_group_747","747","Management and credit insurance fees","Förvaltnings- och kreditförsäkringsavgifter"
"se_group_749","749","Other pension costs","Övriga pensionskostnader"
"se_group_75","75","Social security and other statutory and contractual contributions","Sociala och andra avgifter enligt lag och avtal"
"se_group_750","750","Social security and other statutory and contractual contributions (group account)","Sociala och andra avgifter enligt lag och avtal (gruppkonto)"
"se_group_751","751","Employer's contributions 31.42%","Arbetsgivaravgifter 31,42 %"
"se_group_753","753","Special payroll tax","Särskild löneskatt"
"se_group_755","755","Tax on returns on pension funds","Avkastningsskatt på pensionsmedel"
"se_group_757","757","Labour market insurance premiums","Premier för arbetsmarknadsförsäkringar"
"se_group_758","758","Group insurance premiums","Gruppförsäkringspremier"
"se_group_759","759","Other social and other contributions according to law and agreements","Övriga sociala och andra avgifter enligt lag och avtal"
"se_group_76","76","Other staff costs","Övriga personalkostnader"
"se_group_760","760","Other staff costs (group account)","Övriga personalkostnader (gruppkonto)"
"se_group_761","761","Education and training","Utbildning"
"se_group_762","762","Sickness and health care","Sjuk- och hälsovård"
"se_group_763","763","Staff representation","Personalrepresentation"
"se_group_765","765","Sick pay insurance","Sjuklöneförsäkring"
"se_group_767","767","Change in staff foundation capital","Förändring av personalstiftelsekapital"
"se_group_769","769","Other staff costs","Övriga personalkostnader"
"se_group_77","77","Impairment losses and reversal of impairment losses","Nedskrivningar och återföring av nedskrivningar"
"se_group_771","771","Impairment of intangible fixed assets","Nedskrivningar av immateriella anläggningstillgångar"
"se_group_772","772","Impairment of land and buildings","Nedskrivningar av byggnader och mark"
"se_group_773","773","Impairment of machinery and equipment","Nedskrivningar av maskiner och inventarier"
"se_group_774","774","Impairment of certain current assets","Nedskrivningar av vissa omsättningstillgångar"
"se_group_776","776","Reversal of impairment losses on intangible assets","Återföring av nedskrivningar av immateriella anläggningstillgångar"
"se_group_777","777","Reversal of impairment losses on land and buildings","Återföring av nedskrivningar av byggnader och mark"
"se_group_778","778","Reversal of impairment losses on machinery and equipment","Återföring av nedskrivningar av maskiner och inventarier"
"se_group_779","779","Reversal of impairment losses on certain current assets","Återföring av nedskrivningar av vissa omsättningstillgångar"
"se_group_78","78","Depreciation according to plan","Avskrivningar enligt plan"
"se_group_781","781","Amortisation of intangible fixed assets","Avskrivningar på immateriella anläggningstillgångar"
"se_group_782","782","Depreciation of buildings and land improvements","Avskrivningar på byggnader och markanläggningar"
"se_group_783","783","Depreciation of machinery and equipment","Avskrivningar på maskiner och inventarier"
"se_group_784","784","Depreciation of improvements to property owned by others","Avskrivningar på förbättringsutgifter på annans fastighet"
"se_group_79","79","Other operating expenses","Övriga rörelsekostnader"
"se_group_794","794","Unrealised positive/negative change in value of hedging instruments","Orealiserade positiva/negativa värdeförändringar på säkringsinstrument"
"se_group_796","796","Exchange rate losses on operating receivables and liabilities","Valutakursförluster på fordringar och skulder av rörelsekaraktär"
"se_group_797","797","Loss on disposal of intangible and tangible fixed assets","Förlust vid avyttring av immateriella och materiella anläggningstillgångar"
"se_group_799","799","Other operating expenses","Övriga rörelsekostnader"
"se_group_8","8","Financial and other income/revenue and expenditure/expenses","Finansiella och andra inkomster/intäkter och utgifter/kostnader"
"se_group_80","80","Income from shares in group companies","Resultat från andelar i koncernföretag"
"se_group_801","801","Dividends on shares in group companies","Utdelning på andelar i koncernföretag"
"se_group_802","802","Profit/loss on sale of shares in group companies","Resultat vid försäljning av andelar i koncernföretag"
"se_group_803","803","Share of profits of partnerships (subsidiaries)","Resultatandelar från handelsbolag (dotterföretag)"
"se_group_807","807","Impairment losses on shares in and long-term receivables from group companies","Nedskrivningar av andelar i och långfristiga fordringar hos koncernföretag"
"se_group_808","808","Reversals of impairment losses on investments in and long-term receivables from group companies","Återföringar av nedskrivningar av andelar i och långfristiga fordringar hos koncernföretag"
"se_group_81","81","Income from investments in associates","Resultat från andelar i intresseföretag"
"se_group_811","811","Dividends on shares in associates, jointly controlled entities and other entities in which there is an ownership interest","Utdelningar på andelar i intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"se_group_812","812","Gains on the sale of shares in associates, jointly controlled entities and other entities in which a participation is held","Resultat vid försäljning av andelar i intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"se_group_813","813","Profit shares from partnerships (associates, jointly controlled entities and other entities in which there is a participating interest)","Resultatandelar från handelsbolag (intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i)"
"se_group_817","817","Impairment losses on investments and long-term receivables of associates, jointly controlled entities and other entities in which an ownership interest is held","Nedskrivningar av andelar i och långfristiga fordringar hos intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i"
"se_group_818","818","Reversals of impairment losses on investments in and long-term receivables from associates","Återföringar av nedskrivningar av andelar i och långfristiga fordringar hos intresseföretag"
"se_group_82","82","Income from other securities and long-term receivables","Resultat från övriga värdepapper och långfristiga fordringar"
"se_group_821","821","Dividends on shares in other enterprises","Utdelningar på andelar i andra företag"
"se_group_822","822","Gains on the sale of securities and long-term receivables from other enterprises","Resultat vid försäljning av värdepapper i och långfristiga fordringar hos andra företag"
"se_group_823","823","Exchange rate differences on long-term receivables","Valutakursdifferenser på långfristiga fordringar"
"se_group_824","824","Share of profits of partnerships (other enterprises)","Resultatandelar från handelsbolag (andra företag)"
"se_group_825","825","Interest income on long-term receivables from and securities of other enterprises","Ränteintäkter från långfristiga fordringar hos och värdepapper i andra företag"
"se_group_826","826","Interest income on long-term receivables from group companies","Ränteintäkter från långfristiga fordringar hos koncernföretag"
"se_group_827","827","Impairment losses on holdings of shares and long-term receivables from other corporations","Nedskrivningar av innehav av andelar i och långfristiga fordringar hos andra företag"
"se_group_828","828","Reversals of impairment losses on investments in and long-term receivables from other enterprises","Återföringar av nedskrivningar av andelar i och långfristiga fordringar hos andra företag"
"se_group_829","829","Fair value measurement, fixed assets","Värdering till verkligt värde, anläggningstillgångar"
"se_group_83","83","Other interest income and similar income items","Övriga ränteintäkter och liknande resultatposter"
"se_group_831","831","Interest income from current assets","Ränteintäkter från omsättningstillgångar"
"se_group_832","832","Fair value measurement, current assets","Värdering till verkligt värde, omsättningstillgångar"
"se_group_833","833","Exchange rate differences on short-term receivables and investments","Valutakursdifferenser på kortfristiga fordringar och placeringar"
"se_group_834","834","Dividends on short-term investments","Utdelningar på kortfristiga placeringar"
"se_group_835","835","Gain on sale of short-term investments","Resultat vid försäljning av kortfristiga placeringar"
"se_group_836","836","Other interest income from group companies","Övriga ränteintäkter från koncernföretag"
"se_group_837","837","Impairment losses on short-term investments","Nedskrivningar av kortfristiga placeringar"
"se_group_838","838","Reversals of impairment losses on short-term investments","Återföringar av nedskrivningar av kortfristiga placeringar"
"se_group_839","839","Other financial income","Övriga finansiella intäkter"
"se_group_84","84","Interest expense and similar items","Räntekostnader och liknande resultatposter"
"se_group_840","840","Interest expense (group account)","Räntekostnader (gruppkonto)"
"se_group_841","841","Interest expense on long-term liabilities","Räntekostnader för långfristiga skulder"
"se_group_842","842","Interest expense on short-term liabilities","Räntekostnader för kortfristiga skulder"
"se_group_843","843","Exchange rate differences on liabilities","Valutakursdifferenser på skulder"
"se_group_844","844","Interest subsidies received","Erhållna räntebidrag"
"se_group_845","845","Unrealised changes in value of liabilities","Orealiserade värdeförändringar på skulder"
"se_group_846","846","Interest expenses to group companies","Räntekostnader till koncernföretag"
"se_group_848","848","Capitalised interest expenses","Aktiverade ränteutgifter"
"se_group_849","849","Other debt-related items","Övriga skuldrelaterade poster"
"se_group_85","85","Free account group","Fri kontogrupp"
"se_group_86","86","Unrestricted account group","Fri kontogrupp"
"se_group_87","87","Unrestricted account group","Fri kontogrupp"
"se_group_88","88","Appropriations","Bokslutsdispositioner"
"se_group_881","881","Change in tax allocation reserve","Förändring av periodiseringsfond"
"se_group_882","882","Group contributions received","Mottagna koncernbidrag"
"se_group_883","883","Group contributions paid","Lämnade koncernbidrag"
"se_group_884","884","Indemnities paid","Lämnade gottgörelser"
"se_group_885","885","Change in excess depreciation","Förändring av överavskrivningar"
"se_group_886","886","Change in compensation fund","Förändring av ersättningsfond"
"se_group_889","889","Other appropriations","Övriga bokslutsdispositioner"
"se_group_89","89","Taxes and profit for the year","Skatter och årets resultat"
"se_group_891","891","Taxes charged to the profit for the year","Skatt som belastar årets resultat"
"se_group_892","892","Tax due to change in taxation","Skatt på grund av ändrad beskattning"
"se_group_893","893","Tax refunded","Restituerad skatt"
"se_group_894","894","Deferred tax","Uppskjuten skatt"
"se_group_898","898","Other taxes","Övriga skatter"
"se_group_899","899","Profit or loss","Resultat"

```

## File: data\template\account.tax-se.csv

```csv
"id","name","description","invoice_label","amount","type_tax_use","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@sv_SE"
"sale_tax_25_goods","25% G","VAT Sale of goods 25%","25%","25.0","sale","tax_group_25","base","invoice","+se_05","","","Utgående moms 25%"
"","","","","","","","tax","invoice","+se_10","a2611","",""
"","","","","","","","base","refund","-se_05","","",""
"","","","","","","","tax","refund","-se_10","a2611","",""
"sale_tax_25_services","25% S","VAT Sale of services 25%","25%","25.0","sale","tax_group_25","base","invoice","+se_05","","","Utgående moms Tjänst 25%"
"","","","","","","","tax","invoice","+se_10","a2611","",""
"","","","","","","","base","refund","-se_05","","",""
"","","","","","","","tax","refund","-se_10","a2611","",""
"purchase_tax_25_goods","25% G","VAT Purchases of goods 25%","25%","25.0","purchase","tax_group_25","base","invoice","","","","Ingående moms 25%"
"","","","","","","","tax","invoice","+se_48","a2641","",""
"","","","","","","","base","refund","","","",""
"","","","","","","","tax","refund","-se_48","a2641","",""
"purchase_tax_25_services","25% S","VAT Purchases of services 25%","25%","25.0","purchase","tax_group_25","base","invoice","","","","Ingående moms Tjänst 25%"
"","","","","","","","tax","invoice","+se_48","a2641","",""
"","","","","","","","base","refund","","","",""
"","","","","","","","tax","refund","-se_48","a2641","",""
"sale_tax_12_goods","12% G","VAT Sale of goods 12%","12%","12.0","sale","tax_group_12","base","invoice","+se_05","","","Utgående moms 12%"
"","","","","","","","tax","invoice","+se_11","a2621","",""
"","","","","","","","base","refund","-se_05","","",""
"","","","","","","","tax","refund","-se_11","a2621","",""
"sale_tax_12_services","12% S","VAT Sale of services 12%","12%","12.0","sale","tax_group_12","base","invoice","+se_05","","","Utgående moms Tjänst 12%"
"","","","","","","","tax","invoice","+se_11","a2621","",""
"","","","","","","","base","refund","-se_05","","",""
"","","","","","","","tax","refund","-se_11","a2621","",""
"purchase_tax_12_goods","12% G","VAT Purchases of goods 12%","12%","12.0","purchase","tax_group_12","base","invoice","","","","Ingående moms 12%"
"","","","","","","","tax","invoice","+se_48","a2641","",""
"","","","","","","","base","refund","","","",""
"","","","","","","","tax","refund","-se_48","a2641","",""
"purchase_tax_12_services","12% S","VAT Purchases of services 12%","12%","12.0","purchase","tax_group_12","base","invoice","","","","Ingående moms Tjänst 12%"
"","","","","","","","tax","invoice","+se_48","a2641","",""
"","","","","","","","base","refund","","","",""
"","","","","","","","tax","refund","-se_48","a2641","",""
"sale_tax_6_goods","6% G","VAT Sale of goods 6%","6%","6.0","sale","tax_group_6","base","invoice","+se_05","","","Utgående moms 6%"
"","","","","","","","tax","invoice","+se_12","a2631","",""
"","","","","","","","base","refund","-se_05","","",""
"","","","","","","","tax","refund","-se_12","a2631","",""
"sale_tax_6_services","6% S","VAT Sale of services 6%","6%","6.0","sale","tax_group_6","base","invoice","+se_05","","","Utgående moms Tjänst 6%"
"","","","","","","","tax","invoice","+se_12","a2631","",""
"","","","","","","","base","refund","-se_05","","",""
"","","","","","","","tax","refund","-se_12","a2631","",""
"purchase_tax_6_goods","6% G","VAT Purchase of goods 6%","6%","6.0","purchase","tax_group_6","base","invoice","","","","Ingående moms 6%"
"","","","","","","","tax","invoice","+se_48","a2641","",""
"","","","","","","","base","refund","","","",""
"","","","","","","","tax","refund","-se_48","a2641","",""
"purchase_tax_6_services","6% S","VAT Purchase of services 6%","6%","6.0","purchase","tax_group_6","base","invoice","","","","Ingående moms Tjänst 6%"
"","","","","","","","tax","invoice","+se_48","a2641","",""
"","","","","","","","base","refund","","","",""
"","","","","","","","tax","refund","-se_48","a2641","",""
"sale_tax_services_EC","0% EU S","VAT Sale of services in the EU 0%","0%","0.0","sale","tax_group_0","base","invoice","+se_39","","","Momsfri försäljning av tjänst EU"
"","","","","","","","tax","invoice","","","",""
"","","","","","","","base","refund","-se_39","","",""
"","","","","","","","tax","refund","","","",""
"sale_tax_goods_EC","0% EU G","VAT Sale of goods in the EU 0%","0%","0.0","sale","tax_group_0","base","invoice","+se_35","","","Momsfri Försäljning av varor EU"
"","","","","","","","tax","invoice","","","",""
"","","","","","","","base","refund","-se_35","","",""
"","","","","","","","tax","refund","","","",""
"purchase_goods_tax_25_EC","25% EU G","VAT Purchase of goods in the EU 25%","25%","25.0","purchase","tax_group_25","base","invoice","+se_20","","","Inköp av varor EU moms 25%"
"","","","","","","","tax","invoice","+se_30||+se_48","a2645","",""
"","","","","","","","tax","invoice","","a2614","-100",""
"","","","","","","","base","refund","-se_20","","",""
"","","","","","","","tax","refund","-se_30||-se_48","a2645","",""
"","","","","","","","tax","refund","","a2614","-100",""
"purchase_goods_tax_12_EC","12% EU G","VAT Purchase of goods in the EU 12%","12%","12.0","purchase","tax_group_12","base","invoice","+se_20","","","Inköp av varor EU moms 12%"
"","","","","","","","tax","invoice","+se_31||+se_48","a2645","",""
"","","","","","","","tax","invoice","","a2624","-100",""
"","","","","","","","base","refund","-se_20","","",""
"","","","","","","","tax","refund","-se_31||-se_48","a2645","",""
"","","","","","","","tax","refund","","a2624","-100",""
"purchase_goods_tax_6_EC","6% EU G","VAT Purchase of goods in the EU 6%","6%","6.0","purchase","tax_group_6","base","invoice","+se_20","","","Inköp av varor EU moms 6%"
"","","","","","","","tax","invoice","+se_32||+se_48","a2645","",""
"","","","","","","","tax","invoice","","a2634","-100",""
"","","","","","","","base","refund","-se_20","","",""
"","","","","","","","tax","refund","-se_32||-se_48","a2645","",""
"","","","","","","","tax","refund","","a2634","-100",""
"purchase_services_tax_25_EC","25% EU S","VAT Purchase of services in the EU 25%","25%","25.0","purchase","tax_group_25","base","invoice","+se_21","","","Inköp av tjänst EU moms 25%"
"","","","","","","","tax","invoice","+se_30||+se_48","a2645","",""
"","","","","","","","tax","invoice","","a2614","-100",""
"","","","","","","","base","refund","-se_21","","",""
"","","","","","","","tax","refund","-se_30||-se_48","a2645","",""
"","","","","","","","tax","refund","","a2614","-100",""
"purchase_services_tax_12_EC","12% EU S","VAT Purchase of services in the EU 12%","12%","12.0","purchase","tax_group_12","base","invoice","+se_21","","","Inköp av tjänst EU moms 12%"
"","","","","","","","tax","invoice","+se_31||+se_48","a2645","",""
"","","","","","","","tax","invoice","","a2624","-100",""
"","","","","","","","base","refund","-se_21","","",""
"","","","","","","","tax","refund","-se_31||-se_48","a2645","",""
"","","","","","","","tax","refund","","a2624","-100",""
"purchase_services_tax_6_EC","6% EU S","VAT Purchase of services in the EU 6%","6%","6.0","purchase","tax_group_6","base","invoice","+se_21","","","Inköp av tjänst EU moms 6%"
"","","","","","","","tax","invoice","+se_32||+se_48","a2645","",""
"","","","","","","","tax","invoice","","a2634","-100",""
"","","","","","","","base","refund","-se_21","","",""
"","","","","","","","tax","refund","-se_32||-se_48","a2645","",""
"","","","","","","","tax","refund","","a2634","-100",""
"purchase_construction_services_tax_25_EC","25% EU RS","VAT Purchase of services in Sweden, reverse charge 25%","25%","25.0","purchase","tax_group_25","base","invoice","+se_24","","","Inköpta tjänster i Sverige, omvändskattskyldighet, 25 %"
"","","","","","","","tax","invoice","+se_30||+se_48","a2647","",""
"","","","","","","","tax","invoice","","a2614","-100",""
"","","","","","","","base","refund","-se_24","","",""
"","","","","","","","tax","refund","-se_30||-se_48","a2647","",""
"","","","","","","","tax","refund","","a2614","-100",""
"purchase_construction_services_tax_12_EC","12% EU RS","VAT Purchase of services in Sweden, reverse charge 12%","12%","12.0","purchase","tax_group_12","base","invoice","+se_24","","","Inköpta tjänster i Sverige, omvändskattskyldighet, 12 %"
"","","","","","","","tax","invoice","+se_31||+se_48","a4426","",""
"","","","","","","","tax","invoice","","a2624","-100",""
"","","","","","","","base","refund","-se_24","","",""
"","","","","","","","tax","refund","-se_31||-se_48","a4426","",""
"","","","","","","","tax","refund","","a2624","-100",""
"purchase_construction_services_tax_6_EC","6% EU RS","VAT Purchase of services in Sweden, reverse charge 6%","6%","6.0","purchase","tax_group_6","base","invoice","+se_24","","","Inköpta tjänster i Sverige, omvändskattskyldighet, 6 %"
"","","","","","","","tax","invoice","+se_32||+se_48","a4427","",""
"","","","","","","","tax","invoice","","a2634","-100",""
"","","","","","","","base","refund","-se_24","","",""
"","","","","","","","tax","refund","-se_32||-se_48","a4427","",""
"","","","","","","","tax","refund","","a2634","-100",""
"sale_tax_services_NEC","0% EU RS","VAT Sale of services outside EU 0%","0%","0.0","sale","tax_group_0","base","invoice","+se_39","","","Momsfri försäljning av tjänst utanför EU"
"","","","","","","","tax","invoice","","","",""
"","","","","","","","base","refund","-se_39","","",""
"","","","","","","","tax","refund","","","",""
"sale_tax_goods_NEC","0% EX G","VAT Sale of goods outside EU 0%","0%","0.0","sale","tax_group_0","base","invoice","+se_36","","","Momsfri försäljning av varor utanför EU"
"","","","","","","","tax","invoice","","","",""
"","","","","","","","base","refund","-se_36","","",""
"","","","","","","","tax","refund","","","",""
"purchase_goods_tax_25_NEC","25% EX G","VAT Purchase of goods outside EU 25%","25%","25.0","purchase","tax_group_25","base","invoice","+se_50","","","Beskattningsunderlag vid import 25%"
"","","","","","","","tax","invoice","+se_60","a2645","",""
"","","","","","","","tax","invoice","","a2615","-100",""
"","","","","","","","base","refund","-se_50","","",""
"","","","","","","","tax","refund","-se_60","a2645","",""
"","","","","","","","tax","refund","","a2615","-100",""
"purchase_goods_tax_12_NEC","12% EX G","VAT Purchase of goods outside EU 12%","12%","12.0","purchase","tax_group_12","base","invoice","+se_50","","","Beskattningsunderlag vid import 12%"
"","","","","","","","tax","invoice","+se_61","a2645","",""
"","","","","","","","tax","invoice","","a2625","-100",""
"","","","","","","","base","refund","-se_50","","",""
"","","","","","","","tax","refund","-se_61","a2645","",""
"","","","","","","","tax","refund","","a2625","-100",""
"purchase_goods_tax_6_NEC","6% EX G","VAT Purchase of goods outside EU 6%","6%","6.0","purchase","tax_group_6","base","invoice","+se_50","","","Beskattningsunderlag vid import 6%"
"","","","","","","","tax","invoice","+se_62","a2645","",""
"","","","","","","","tax","invoice","","a2635","-100",""
"","","","","","","","base","refund","-se_50","","",""
"","","","","","","","tax","refund","-se_62","a2645","",""
"","","","","","","","tax","refund","","a2635","-100",""
"purchase_services_tax_25_NEC","25% EX S","VAT Purchase of services outside EU 25%","25%","25.0","purchase","tax_group_25","base","invoice","+se_22","","","Inköp av tjänster utanför EU 25%"
"","","","","","","","tax","invoice","+se_30||+se_48","a2645","",""
"","","","","","","","tax","invoice","","a2614","-100",""
"","","","","","","","base","refund","-se_22","","",""
"","","","","","","","tax","refund","-se_30||-se_48","a2645","",""
"","","","","","","","tax","refund","","a2614","-100",""
"purchase_services_tax_12_NEC","12% EX S","VAT Purchase of services outside EU 12%","12%","12.0","purchase","tax_group_12","base","invoice","+se_22","","","Inköp av tjänster utanför EU 12%"
"","","","","","","","tax","invoice","+se_31||+se_48","a2645","",""
"","","","","","","","tax","invoice","","a2624","-100",""
"","","","","","","","base","refund","-se_22","","",""
"","","","","","","","tax","refund","-se_31||-se_48","a2645","",""
"","","","","","","","tax","refund","","a2624","-100",""
"purchase_services_tax_6_NEC","6% EX S","VAT Purchase of services outside EU 6%","6%","6.0","purchase","tax_group_6","base","invoice","+se_22","","","Inköp av tjänster utanför EU 6%"
"","","","","","","","tax","invoice","+se_32||+se_48","a2645","",""
"","","","","","","","tax","invoice","","a2634","-100",""
"","","","","","","","base","refund","-se_22","","",""
"","","","","","","","tax","refund","-se_32||-se_48","a2645","",""
"","","","","","","","tax","refund","","a2634","-100",""
"triangular_tax_25_goods","25% EU G Tr","VAT Triangular Sale of goods 25%","25%","25.0","sale","tax_group_25","base","invoice","+se_37||-se_38","","","Trepartshandel - moms 25%"
"","","","","","","","tax","invoice","","a2615","",""
"","","","","","","","tax","invoice","","a2645","-100",""
"","","","","","","","base","refund","-se_37||+se_38","","",""
"","","","","","","","tax","refund","","a2615","",""
"","","","","","","","tax","refund","","a2645","-100",""
"triangular_tax_12_goods","12% EU G Tr","VAT Triangular Sale of goods 12%","12%","12.0","sale","tax_group_12","base","invoice","+se_37||-se_38","","","Trepartshandel - moms 12%"
"","","","","","","","tax","invoice","","a2625","",""
"","","","","","","","tax","invoice","","a2645","-100",""
"","","","","","","","base","refund","-se_37||+se_38","","",""
"","","","","","","","tax","refund","","a2625","",""
"","","","","","","","tax","refund","","a2645","-100",""
"triangular_tax_6_goods","6% EU G Tr","VAT Triangular Sale of goods 6%","6%","6.0","sale","tax_group_6","base","invoice","+se_37||-se_38","","","Trepartshandel - moms 6%"
"","","","","","","","tax","invoice","","a2635","",""
"","","","","","","","tax","invoice","","a2645","-100",""
"","","","","","","","base","refund","-se_37||+se_38","","",""
"","","","","","","","tax","refund","","a2635","",""
"","","","","","","","tax","refund","","a2645","-100",""
"triangular_tax_0_goods","0% EU G Tr","VAT Triangular Sale of goods 0%","0%","0.0","sale","tax_group_0","base","invoice","+se_37||-se_38","","","Trepartshandel - momsfrei"
"","","","","","","","tax","invoice","","","",""
"","","","","","","","base","refund","-se_37||+se_38","","",""
"","","","","","","","tax","refund","","","",""

```

## File: data\template\account.tax.group-se.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id","pos_receipt_label","name@sv_SE"
"tax_group_25","VAT 25%","base.se","a2650","a1650","A","Moms 25%"
"tax_group_12","VAT 12%","base.se","a2650","a1650","B","Moms 12%"
"tax_group_6","VAT 6%","base.se","a2650","a1650","C","Moms 6%"
"tax_group_0","VAT 0%","base.se","a2650","a1650","D","Moms 0%"

```

## File: data\template\account.tax.group-se_K2.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id","name@sv_SE"
"tax_group_25","VAT 25%","base.se","a2650","a1650","Moms 25%"
"tax_group_12","VAT 12%","base.se","a2650","a1650","Moms 12%"
"tax_group_6","VAT 6%","base.se","a2650","a1650","Moms 6%"
"tax_group_0","VAT 0%","base.se","a2650","a1650","Moms 0%"

```

## File: data\template\account.tax.group-se_K3.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id","name@sv_SE"
"tax_group_25","VAT 25%","base.se","a2650","a1650","Moms 25%"
"tax_group_12","VAT 12%","base.se","a2650","a1650","Moms 12%"
"tax_group_6","VAT 6%","base.se","a2650","a1650","Moms 6%"
"tax_group_0","VAT 0%","base.se","a2650","a1650","Moms 0%"

```

## File: migrations\1.1\end-migrate.py

```python
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'se')], order="parent_path"):
        env['account.chart.template'].try_loading('se', company)

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from odoo.exceptions import ValidationError


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    invoice_reference_model = fields.Selection(selection_add=[('se_ocr2', 'Sweden OCR Level 1 & 2'), ('se_ocr3', 'Sweden OCR Level 3'), ('se_ocr4', 'Sweden OCR Level 4')], ondelete={'se_ocr2': 'set default', 'se_ocr3': 'set default', 'se_ocr4': 'set default'})
    l10n_se_invoice_ocr_length = fields.Integer(string='OCR Number Length', help="Total length of OCR Reference Number including checksum.", default=6)

    @api.constrains('l10n_se_invoice_ocr_length')
    def _check_l10n_se_invoice_ocr_length(self):
        for journal in self:
            if journal.l10n_se_invoice_ocr_length < 6:
                raise ValidationError(_('OCR Reference Number length need to be greater than 5. Please correct settings under invoice journal settings.'))

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from stdnum import luhn


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _get_invoice_reference_se_ocr2(self, reference):
        self.ensure_one()
        return reference + luhn.calc_check_digit(reference)

    def _get_invoice_reference_se_ocr3(self, reference):
        self.ensure_one()
        reference = reference + str(len(reference) + 2)[:1]
        return reference + luhn.calc_check_digit(reference)

    def _get_invoice_reference_se_ocr4(self, reference):
        self.ensure_one()

        ocr_length = self.journal_id.l10n_se_invoice_ocr_length

        if len(reference) + 1 > ocr_length:
            raise UserError(_("OCR Reference Number length is greater than allowed. Allowed length in invoice journal setting is %s.", ocr_length))

        reference = reference.rjust(ocr_length - 1, '0')
        return reference + luhn.calc_check_digit(reference)


    def _get_invoice_reference_se_ocr2_invoice(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr2(str(self.id))

    def _get_invoice_reference_se_ocr3_invoice(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr3(str(self.id))

    def _get_invoice_reference_se_ocr4_invoice(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr4(str(self.id))

    def _get_invoice_reference_se_ocr2_partner(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr2(self.partner_id.ref if str(self.partner_id.ref).isdecimal() else str(self.partner_id.id))

    def _get_invoice_reference_se_ocr3_partner(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr3(self.partner_id.ref if str(self.partner_id.ref).isdecimal() else str(self.partner_id.id))

    def _get_invoice_reference_se_ocr4_partner(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr4(self.partner_id.ref if str(self.partner_id.ref).isdecimal() else str(self.partner_id.id))

    @api.onchange('partner_id')
    def _onchange_partner_id(self):
        """ If Vendor Bill and Vendor OCR is set, add it. """
        if self.partner_id and self.move_type == 'in_invoice' and self.partner_id.l10n_se_default_vendor_payment_ref:
            self.payment_reference = self.partner_id.l10n_se_default_vendor_payment_ref
        return super(AccountMove, self)._onchange_partner_id()

    @api.constrains('payment_reference', 'state')
    def _l10n_se_check_payment_reference(self):
        for invoice in self:
            if (
                (invoice.payment_reference or invoice.state == 'posted')
                and invoice.partner_id
                and invoice.move_type == 'in_invoice'
                and invoice.partner_id.l10n_se_check_vendor_ocr
                and invoice.country_code == 'SE'
            ):
                try:
                    luhn.validate(invoice.payment_reference)
                except Exception:
                    raise ValidationError(_("Vendor require OCR Number as payment reference. Payment reference isn't a valid OCR Number."))

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
import re


class ResCompany(models.Model):
    _inherit = 'res.company'

    org_number = fields.Char(compute='_compute_org_number')

    @api.depends('vat')
    def _compute_org_number(self):
        for company in self:
            if company.account_fiscal_country_id.code == "SE" and company.vat:
                org_number = re.sub(r'\D', '', company.vat)[:-2]
                org_number = org_number[:6] + '-' + org_number[6:]

                company.org_number = org_number
            else:
                company.org_number = ''

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from stdnum import luhn


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_se_check_vendor_ocr = fields.Boolean(string='Check Vendor OCR', help='This Vendor uses OCR Number on their Vendor Bills.')
    l10n_se_default_vendor_payment_ref = fields.Char(string='Default Vendor Payment Ref', help='If set, the vendor uses the same Default Payment Reference or OCR Number on all their Vendor Bills.')

    @api.onchange('l10n_se_default_vendor_payment_ref')
    def onchange_l10n_se_default_vendor_payment_ref(self):
        if not self.l10n_se_default_vendor_payment_ref == "" and self.l10n_se_check_vendor_ocr:
            reference = self.l10n_se_default_vendor_payment_ref
            try:
                luhn.validate(reference)
            except: 
                return {'warning': {'title': _('Warning'), 'message': _('Default vendor OCR number isn\'t a valid OCR number.')}}

```

## File: models\template_se.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('se')
    def _get_se_template_data(self):
        return {
            'property_account_receivable_id': 'a1510',
            'property_account_payable_id': 'a2440',
            'property_account_expense_categ_id': 'a4000',
            'property_account_income_categ_id': 'a3001',
            'property_stock_account_input_categ_id': 'a4960',
            'property_stock_account_output_categ_id': 'a4960',
            'property_stock_valuation_account_id': 'a1410',
            'code_digits': '4',
        }

    @template('se', 'res.company')
    def _get_se_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.se',
                'bank_account_code_prefix': '193',
                'cash_account_code_prefix': '191',
                'transfer_account_code_prefix': '194',
                'account_default_pos_receivable_account_id': 'a1910',
                'income_currency_exchange_account_id': 'a3960',
                'expense_currency_exchange_account_id': 'a3960',
                'account_journal_early_pay_discount_loss_account_id': 'a9993',
                'account_journal_early_pay_discount_gain_account_id': 'a9994',
                'account_sale_tax_id': 'sale_tax_25_goods',
                'account_purchase_tax_id': 'purchase_tax_25_goods',
            },
        }

```

## File: models\template_se_K2.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('se_K2')
    def _get_se_K2_template_data(self):
        return {
            'name': 'Swedish BAS Chart of Account complete K2',
            'parent': 'se',
            'code_digits': '4',
        }

    @template('se_K2', 'res.company')
    def _get_se_K2_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.se',
                'bank_account_code_prefix': '193',
                'cash_account_code_prefix': '191',
                'transfer_account_code_prefix': '194',
            },
        }

```

## File: models\template_se_K3.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('se_K3')
    def _get_se_K3_template_data(self):
        return {
            'name': 'Swedish BAS Chart of Account complete K3',
            'parent': 'se_K2',
            'code_digits': '4',
        }

    @template('se_K3', 'res.company')
    def _get_se_K3_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.se',
                'bank_account_code_prefix': '193',
                'cash_account_code_prefix': '191',
                'transfer_account_code_prefix': '194',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_se_K2
from . import template_se_K3
from . import template_se
from . import res_company
from . import account_move
from . import account_journal
from . import res_partner

```

## File: views\account_journal_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_account_journal_se_ocr_form" model="ir.ui.view">
            <field name="name">account.journal.se.ocr.form</field>
            <field name="model">account.journal</field>
            <field name="inherit_id" ref="account.view_account_journal_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='invoice_reference_model']" position="after">
                    <field name="l10n_se_invoice_ocr_length" invisible="invoice_reference_model != 'se_ocr4'"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\partner_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_partner_ocr_form" model="ir.ui.view">
            <field name="name">res.partner.ocr.form</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="account.view_partner_property_form"/>
            <field name="arch" type="xml">
                <group name="accounting_entries" position="after">
                    <group string="Payment Options Sweden" name="payment_options" invisible="'SE' not in fiscal_country_codes">
                        <field name="l10n_se_check_vendor_ocr"/>
                        <field name="l10n_se_default_vendor_payment_ref"/>
                    </group>
                </group>
            </field>
        </record>
    </data>
</odoo>

```

