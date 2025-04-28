# Odoo Module: l10n_mn

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Mongolia - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'version': '1.1',
    'icon': '/account/static/description/l10n.png',
    'countries': ['mn'],
    'category': 'Accounting/Localizations/Account Charts',
    'author': 'BumanIT LLC, Odoo S.A.',
    'description': """
This is the module to manage the accounting chart for Mongolia.
===============================================================

    * the Mongolia Official Chart of Accounts,
    * the Tax Code Chart for Mongolia
    * the main taxes used in Mongolia

Financial requirement contributor: Baskhuu Lodoikhuu. BumanIT LLC
""",
    'depends': [
        'account',
    ],
    'data': [
        'data/account.account.tag.csv',
        'data/vat_report.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.tag.csv

```csv
id,name,applicability,country_id:id
account_cashflow_tag_111,"CF:[1.1.1] Income from the sale of goods and services",accounts,
account_cashflow_tag_112,"CF:[1.1.2] Revenue from royalties, fees and charges",accounts,
account_cashflow_tag_113,"CF:[1.1.3] Amount received from insured spouse",accounts,
account_cashflow_tag_114,"CF:[1.1.4] Tax refunds",accounts,
account_cashflow_tag_115,"CF:[1.1.5] Subsidy and financing income",accounts,
account_cashflow_tag_116,"CF:[1.1.6] Other cash income",accounts,
account_cashflow_tag_121,"CF:[1.2.1] Paid to employees",accounts,
account_cashflow_tag_122,"CF:[1.2.2] Paid to Social Security",accounts,
account_cashflow_tag_123,"CF:[1.2.3] Paid to purchase inventory",accounts,
account_cashflow_tag_124,"CF:[1.2.4] Paid for operating expenses",accounts,
account_cashflow_tag_125,"CF: [1.2.5] Paid for fuel, transportation, spare parts",accounts,
account_cashflow_tag_126,"CF:[1.2.6] Interest paid",accounts,
account_cashflow_tag_127,"CF:[1.2.7] Paid to tax authorities",accounts,
account_cashflow_tag_128,"CF:[1.2.8] Paid for insurance",accounts,
account_cashflow_tag_129,"CF:[1.2.9] Other cash expenses",accounts,
account_cashflow_tag_211_221,"CF:[2.1.1] Income from sale of fixed assets",accounts,
account_cashflow_tag_212_222,"CF:[2.1.2] Income from sale of intangible assets",accounts,
account_cashflow_tag_213_223,"CF:[2.1.3] Income from sale of investment",accounts,
account_cashflow_tag_214_224,"CF:[2.1.4] Income from sale of other long-term assets",accounts,
account_cashflow_tag_215_225,"CF:[2.1.5] Repayment of loans and cash advances to others",accounts,
account_cashflow_tag_216,"CF:[2.1.6] Interest income received",accounts,
account_cashflow_tag_217,"CF:[2.1.7] Dividends received",accounts,
account_cashflow_tag_221,"CF:[2.2.1] Paid for acquisition and possession of fixed assets",accounts,
account_cashflow_tag_222,"CF:[2.2.2] Paid to acquire intangible assets",accounts,
account_cashflow_tag_223,"CF:[2.2.3] Paid to acquire investment",accounts,
account_cashflow_tag_224,"CF:[2.2.4] Paid to acquire and hold other long-term assets",accounts,
account_cashflow_tag_225,"CF:[2.2.5] Loans and advances to others",accounts,
account_cashflow_tag_311_321,"CF:[3.1.1] Received from borrowing and issuing debt securities",accounts,
account_cashflow_tag_312_323,"CF:[3.1.2] Received from the issuance of shares and other equity securities",accounts,
account_cashflow_tag_313,"CF:[3.1.3] Miscellaneous donations",accounts,
account_cashflow_tag_321,"CF:[3.2.1] Money paid for loans and debt securities",accounts,
account_cashflow_tag_322,"CF:[3.2.2] Paid for finance leases",accounts,
account_cashflow_tag_323,"CF:[3.2.3] Paid to buy back shares",accounts,
account_cashflow_tag_324,"CF:[3.2.4] Dividends paid",accounts,
account_cashflow_tag_325,"CF:[3.2.5] Various donations, grants and fines paid",accounts,
account_cashflow_tag_400,"CF:[4] Exchange rate difference",accounts,

```

## File: data\vat_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="account_report_vat_report" model="account.report">
        <field name="name">VAT Repayment Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.mn"/>
        <field name="filter_multi_company">tax_units</field>
        <field name="column_ids">
            <record id="account_report_vat_report_column" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="vat_report_lineA" model="account.report.line">
                <field name="name">A. CALCULATION OF VAT ON GOODS, WORKS AND SERVICES SOLD FOR THAT MONTH</field>
                <field name="code">VAT_LINEA</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="vat_report_line1" model="account.report.line">
                        <field name="name">Total sales revenue for that month (1=2+3)</field>
                        <field name="code">VAT_LINE1</field>
                        <field name="aggregation_formula">VAT_LINE2.balance + VAT_LINE3.balance</field>
                    </record>
                    <record id="vat_report_line2" model="account.report.line">
                        <field name="name">VAT-exempt sales for the month specified in Article 13 of the Law (2)</field>
                        <field name="code">VAT_LINE2</field>
                        <field name="expression_ids">
                            <record id="vat_report_line2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TT3a[2]</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line3" model="account.report.line">
                        <field name="name">Total revenue from sales of goods, works and services subject to value added tax (3=4+10+29)</field>
                        <field name="code">VAT_LINE3</field>
                        <field name="aggregation_formula">VAT_LINE4.balance + VAT_LINE10.balance + VAT_LINE29.balance</field>
                    </record>
                    <record id="vat_report_line4" model="account.report.line">
                        <field name="name">Income from sales of goods subject to VAT (4=5+...+9)</field>
                        <field name="code">VAT_LINE4</field>
                        <field name="aggregation_formula">VAT_LINE5.balance + VAT_LINE6.balance + VAT_LINE7.balance + VAT_LINE8.balance + VAT_LINE9.balance</field>
                        <field name="children_ids">
                            <record id="vat_report_line5" model="account.report.line">
                                <field name="name">Revenue from sales of goods sold in the domestic market (5)</field>
                                <field name="code">VAT_LINE5</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[5]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line6" model="account.report.line">
                                <field name="name">Revenue from sales of other goods (6)</field>
                                <field name="code">VAT_LINE6</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line6_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[6]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line7" model="account.report.line">
                                <field name="name">Revenue from the sale of rights (7)</field>
                                <field name="code">VAT_LINE7</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[7]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line8" model="account.report.line">
                                <field name="name">Proceeds from retained goods on liquidation (8)</field>
                                <field name="code">VAT_LINE8</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[8]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line9" model="account.report.line">
                                <field name="name">Income from goods transferred for debt payment (9)</field>
                                <field name="code">VAT_LINE9</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line9_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[9]</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line10" model="account.report.line">
                        <field name="name">Income from work and services per VAT (10=11+...+24)</field>
                        <field name="code">VAT_LINE10</field>
                        <field name="aggregation_formula">VAT_LINE11.balance + VAT_LINE12.balance + VAT_LINE13.balance + VAT_LINE14.balance + VAT_LINE15.balance + VAT_LINE16.balance + VAT_LINE17.balance + VAT_LINE18.balance + VAT_LINE19.balance + VAT_LINE20.balance + VAT_LINE21.balance + VAT_LINE22.balance + VAT_LINE23.balance + VAT_LINE24.balance</field>
                        <field name="children_ids">
                            <record id="vat_report_line11" model="account.report.line">
                                <field name="name">Income from performing work and providing services calculated for debt payment (11)</field>
                                <field name="code">VAT_LINE11</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line11_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[11]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line12" model="account.report.line">
                                <field name="name">Income from electricity, heat, gas, water supply, sewerage, postal, telecommunications and other services (12)</field>
                                <field name="code">VAT_LINE12</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line12_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[12]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line13" model="account.report.line">
                                <field name="name">Income from services for leasing, owning and using goods in other forms (13)</field>
                                <field name="code">VAT_LINE13</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line13_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[13]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line14" model="account.report.line">
                                <field name="name">Income from services for renting, owning and using hotels and similar premises (14)</field>
                                <field name="code">VAT_LINE14</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line14_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[14]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line15" model="account.report.line">
                                <field name="name">Income from rental and ownership of immovable and movable property (15)</field>
                                <field name="code">VAT_LINE15</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line15_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[15]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line16" model="account.report.line">
                                <field name="name">Income from the transfer, rental, or sale of inventions, product designs, copyrighted works, trademarks, and know-how (16)</field>
                                <field name="code">VAT_LINE16</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line16_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[16]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line17" model="account.report.line">
                                <field name="name">Income from cash lotteries and paid guessing games (17)</field>
                                <field name="code">VAT_LINE17</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line17_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[17]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line18" model="account.report.line">
                                <field name="name">Income from brokerage services (18)</field>
                                <field name="code">VAT_LINE18</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line18_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[18]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line19" model="account.report.line">
                                <field name="name">Income from interest, fines and penalties received due to wrongful actions of others (19)</field>
                                <field name="code">VAT_LINE19</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line19_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[19]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line20" model="account.report.line">
                                <field name="name">Income from property valuation services (20)</field>
                                <field name="code">VAT_LINE20</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line20_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[20]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line21" model="account.report.line">
                                <field name="name">Budget funding, subsidies, and incentive income provided by the government (21)</field>
                                <field name="code">VAT_LINE21</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line21_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[21]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line22" model="account.report.line">
                                <field name="name">Income from advocacy and legal advice services (22)</field>
                                <field name="code">VAT_LINE22</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line22_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[22]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line23" model="account.report.line">
                                <field name="name">Income from hairdressing, beauty, maintenance, laundry and dry cleaning services (23)</field>
                                <field name="code">VAT_LINE23</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line23_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[23]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line24" model="account.report.line">
                                <field name="name">Income from all types of services other than those specified in Article 13 of the Law (24)</field>
                                <field name="code">VAT_LINE24</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line24_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[24]</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line25" model="account.report.line">
                        <field name="name">Revenue of domestic goods, works and services per VAT for the month (25=4+10)</field>
                        <field name="code">VAT_LINE25</field>
                        <field name="aggregation_formula">VAT_LINE4.balance + VAT_LINE10.balance</field>
                    </record>
                    <record id="vat_report_line26" model="account.report.line">
                        <field name="name">Tax imposed (26=25*10%)</field>
                        <field name="code">VAT_LINE26</field>
                        <field name="aggregation_formula">VAT_LINE25.balance * 0.10</field>
                    </record>
                    <record id="vat_report_line27" model="account.report.line">
                        <field name="name">Income from the sale of goods subject to zero value added tax (27)</field>
                        <field name="code">VAT_LINE27</field>
                        <field name="expression_ids">
                            <record id="vat_report_line27_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TT3a[27]</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line28" model="account.report.line">
                        <field name="name">Income from services subject to 0% value added tax (28)</field>
                        <field name="code">VAT_LINE28</field>
                        <field name="expression_ids">
                            <record id="vat_report_line28_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TT3a[28]</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line29" model="account.report.line">
                        <field name="name">Revenue of export goods, works and services per VAT for the month (29=27+28)</field>
                        <field name="code">VAT_LINE29</field>
                        <field name="aggregation_formula">VAT_LINE27.balance + VAT_LINE28.balance</field>
                    </record>
                    <record id="vat_report_line30" model="account.report.line">
                        <field name="name">Tax imposed (30=29*0%)</field>
                        <field name="code">VAT_LINE30</field>
                        <field name="aggregation_formula">(VAT_LINE27.balance + VAT_LINE28.balance) * 0</field>
                    </record>
                    <record id="vat_report_line31" model="account.report.line">
                        <field name="name">Total VAT tax imposed in that month (31=26+30)</field>
                        <field name="code">VAT_LINE31</field>
                        <field name="aggregation_formula">VAT_LINE26.balance + VAT_LINE30.balance</field>
                    </record>
                </field>
            </record>
            <record id="vat_report_lineB" model="account.report.line">
                <field name="name">B. CALCULATION OF VAT ON GOODS, WORKS AND SERVICES PURCHASED IN THAT MONTH</field>
                <field name="code">VAT_LINEB</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="vat_report_line32" model="account.report.line">
                        <field name="name">Total amount of goods and services purchased in that month (32)</field>
                        <field name="code">VAT_LINE32</field>
                        <field name="expression_ids">
                            <record id="vat_report_line32_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TT3a[32]</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line33" model="account.report.line">
                        <field name="name">The amount of goods, works and services purchased with VAT from BAU purchased in that month (33=34+...+41)</field>
                        <field name="code">VAT_LINE33</field>
                        <field name="aggregation_formula">VAT_LINE34.balance + VAT_LINE35.balance + VAT_LINE36.balance + VAT_LINE37.balance + VAT_LINE38.balance + VAT_LINE39.balance + VAT_LINE40.balance + VAT_LINE41.balance</field>
                        <field name="children_ids">
                            <record id="vat_report_line34" model="account.report.line">
                                <field name="name">Amount of imported goods and services /Amount without VAT/ (34)</field>
                                <field name="code">VAT_LINE34</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line34_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[34]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line35" model="account.report.line">
                                <field name="name">Amount paid for goods, works, and services purchased from the domestic market /Amount without VAT/ (35)</field>
                                <field name="code">VAT_LINE35</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line35_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[35]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line36" model="account.report.line">
                                <field name="name">VAT-free amount of imported goods and services at the time of registration as a withholding tax payer (36)</field>
                                <field name="code">VAT_LINE36</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line36_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[36]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line37" model="account.report.line">
                                <field name="name">Amount without VAT paid for import purchases for the preparation of fixed assets for the month: buildings and structures (37)</field>
                                <field name="code">VAT_LINE37</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line37_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[37]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line38" model="account.report.line">
                                <field name="name">Amount without VAT paid for import purchases for the preparation of fixed assets for the month: machinery and equipment (38)</field>
                                <field name="code">VAT_LINE38</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line38_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[38]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line39" model="account.report.line">
                                <field name="name">Amounts without VAT paid for import purchases for the preparation of fixed assets for the month: other fixed assets (39)</field>
                                <field name="code">VAT_LINE39</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line39_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[39]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line40" model="account.report.line">
                                <field name="name">Amount excluding VAT paid for import purchases for the preparation of fixed assets applicable to the month: exploratory valuation assets (40)</field>
                                <field name="code">VAT_LINE40</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line40_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[40]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line41" model="account.report.line">
                                <field name="name">Amount without VAT of goods and services purchased from livestock and agricultural producers (41)</field>
                                <field name="code">VAT_LINE41</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line41_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[41]</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line42" model="account.report.line">
                        <field name="name">· Total amount of VAT paid on purchased goods, works and services (42=33*10%)</field>
                        <field name="code">VAT_LINE42</field>
                        <field name="groupby" eval="False"/>
                        <field name="foldable" eval="False"/>
                    </record>
                    <record id="vat_report_line42" model="account.report.line">
                        <field name="aggregation_formula">VAT_LINE33.balance * 0.10</field>
                    </record>
                    <record id="vat_report_line43" model="account.report.line">
                        <field name="name">· Amount of non-deductible VAT paid on purchased goods, works and services (43=44+..+48)</field>
                        <field name="code">VAT_LINE43</field>
                        <field name="aggregation_formula">VAT_LINE44.balance + VAT_LINE45.balance + VAT_LINE46.balance + VAT_LINE47.balance + VAT_LINE48.balance</field>
                        <field name="children_ids">
                            <record id="vat_report_line44" model="account.report.line">
                                <field name="name">VAT paid on passenger cars and their spare parts (44)</field>
                                <field name="code">VAT_LINE44</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line44_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[44]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line45" model="account.report.line">
                                <field name="name">VAT paid on goods and services for personal and employee use (45)</field>
                                <field name="code">VAT_LINE45</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line45_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[45]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line46" model="account.report.line">
                                <field name="name">VAT paid on goods and services imported for exempted production and services under Article 13 of the Act (46)</field>
                                <field name="code">VAT_LINE46</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line46_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[46]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line47" model="account.report.line">
                                <field name="name">VAT paid on imported goods and services for pre-use operations (47)</field>
                                <field name="code">VAT_LINE47</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line47_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[47]</field>
                                    </record>
                                </field>
                            </record>
                            <record id="vat_report_line48" model="account.report.line">
                                <field name="name">VAT paid on imported goods and services regardless of taxable goods and services during the reporting period (48)</field>
                                <field name="code">VAT_LINE48</field>
                                <field name="expression_ids">
                                    <record id="vat_report_line48_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TT3a[48]</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line49" model="account.report.line">
                        <field name="name">The amount of VAT to be deducted in that month (49=42-43)</field>
                        <field name="code">VAT_LINE49</field>
                        <field name="aggregation_formula">VAT_LINE42.balance - VAT_LINE43.balance</field>
                    </record>
                </field>
            </record>
            <record id="vat_report_lineC" model="account.report.line">
                <field name="name">C. VAT CALCULATION FOR FINANCIAL LEASE AND FINANCING DEALINGS</field>
                <field name="code">VAT_LINEC</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="vat_report_line50" model="account.report.line">
                        <field name="name">Income from sale of financial lease items in the domestic market under financial lease (50)</field>
                        <field name="code">VAT_LINE50</field>
                        <field name="expression_ids">
                            <record id="vat_report_line50_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TT3a[50]</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line51" model="account.report.line">
                        <field name="name">Revenue from similar transaction services such as factoring and forfeiting (51)</field>
                        <field name="code">VAT_LINE51</field>
                        <field name="foldable" eval="True"/>
                        <field name="expression_ids">
                            <record id="vat_report_line51_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TT3a[51]</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line52" model="account.report.line">
                        <field name="name">Charged VAT (52=(50+51)*10%)</field>
                        <field name="code">VAT_LINE52</field>
                        <field name="groupby" eval="False"/>
                        <field name="foldable" eval="False"/>
                    </record>
                    <record id="vat_report_line52" model="account.report.line">
                        <field name="aggregation_formula">(VAT_LINE50.balance + VAT_LINE51.balance) * 0.10</field>
                    </record>
                    <record id="vat_report_line53" model="account.report.line">
                        <field name="name">Financial Leases Rents paid for domestic market purchases under financial leases (53)</field>
                        <field name="code">VAT_LINE53</field>
                        <field name="expression_ids">
                            <record id="vat_report_line53_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TT3a[53]</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line54" model="account.report.line">
                        <field name="name">Payments made for the purchase of factoring and forfeiting equivalents (54)</field>
                        <field name="code">VAT_LINE54</field>
                        <field name="expression_ids">
                            <record id="vat_report_line54_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TT3a[54]</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line55" model="account.report.line">
                        <field name="name">Deductible VAT (55=(53+54)*10%)</field>
                        <field name="code">VAT_LINE55</field>
                        <field name="aggregation_formula">(VAT_LINE53.balance + VAT_LINE54.balance) * 0.10</field>
                    </record>
                </field>
            </record>
            <record id="vat_report_lineD" model="account.report.line">
                <field name="name">C. OFFICIAL CALCULATION OF VALUE ADDED TAX FOR THE MONTH</field>
                <field name="code">VAT_LINED</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="vat_report_line56" model="account.report.line">
                        <field name="name">VAT payable in that month (56=31+52)</field>
                        <field name="code">VAT_LINE56</field>
                        <field name="aggregation_formula">VAT_LINE31.balance + VAT_LINE52.balance</field>
                    </record>
                    <record id="vat_report_line57" model="account.report.line">
                        <field name="name">VAT to be refunded in that month (57=49+55)</field>
                        <field name="code">VAT_LINE57</field>
                        <field name="aggregation_formula">VAT_LINE49.balance + VAT_LINE55.balance</field>
                    </record>
                </field>
            </record>
            <record id="vat_report_lineE" model="account.report.line">
                <field name="name">E. CORRECTION TO PREVIOUS MONTH'S REPORT</field>
                <field name="code">VAT_LINEE</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="vat_report_line58" model="account.report.line">
                        <field name="name">Sales revenue returns and discounts (58)</field>
                        <field name="code">VAT_LINE58</field>
                        <field name="expression_ids">
                            <record id="vat_report_line58_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TT3a[58]</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line59" model="account.report.line">
                        <field name="name">Paid VAT (59)</field>
                        <field name="code">VAT_LINE59</field>
                        <field name="expression_ids">
                            <record id="vat_report_line59_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TT3a[59]</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line60" model="account.report.line">
                        <field name="name">Purchase returns and discounts (60)</field>
                        <field name="code">VAT_LINE60</field>
                        <field name="expression_ids">
                            <record id="vat_report_line60_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TT3a[60]</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line61" model="account.report.line">
                        <field name="name">Deductible VAT (61)</field>
                        <field name="code">VAT_LINE61</field>
                        <field name="expression_ids">
                            <record id="vat_report_line61_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TT3a[61]</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="vat_report_lineF" model="account.report.line">
                <field name="name">F. OFFICIAL CALCULATION OF VALUE ADDED TAX FOR THE MONTH</field>
                <field name="code">VAT_LINEF</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="vat_report_line62" model="account.report.line">
                        <field name="name">TOTAL PAYMENT APPROPRIATE VAT (62=56+61)</field>
                        <field name="code">VAT_LINE62</field>
                        <field name="aggregation_formula">VAT_LINE56.balance + VAT_LINE61.balance</field>
                    </record>
                    <record id="vat_report_line63" model="account.report.line">
                        <field name="name">TOTAL RECOVERY VAT (63=57+59)</field>
                        <field name="code">VAT_LINE63</field>
                        <field name="aggregation_formula">VAT_LINE57.balance + VAT_LINE59.balance</field>
                    </record>
                    <record id="vat_report_line64" model="account.report.line">
                        <field name="name">VAT TO BE PAYABLE IN THE FINAL CALCULATION (64=62-63)</field>
                        <field name="code">VAT_LINE64</field>
                        <field name="aggregation_formula" eval="False"/>
                        <field name="expression_ids">
                            <record id="vat_report_line64_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">VAT_LINE62.balance - VAT_LINE63.balance</field>
                                <field name="subformula">if_above(MNT(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_report_line65" model="account.report.line">
                        <field name="name">VAT RECOVERED IN FINAL CALCULATION (65=63-62)</field>
                        <field name="code">VAT_LINE65</field>
                        <field name="aggregation_formula" eval="False"/>
                        <field name="expression_ids">
                            <record id="vat_report_line64_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">VAT_LINE63.balance - VAT_LINE62.balance</field>
                                <field name="subformula">if_above(MNT(0))</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-mn.csv

```csv
id,code,name,account_type,tag_ids,reconcile,name@mn
account_template_1001_0101,10010101,Cash,asset_cash,"",0,Бэлэн мөнгө
account_template_1003_0101,10030101,A petty cash fund,asset_cash,"",0,Жижиг мөнгөн сан
account_template_1101_0101,11010101,Money in the bank,asset_cash,"",0,Банкинд байгаа мөнгө
account_template_1103_0101,11030101,Time deposits,asset_cash,"",0,Хугацаагүй хадгаламж
account_template_1104_0101,11040101,Bank payment terminal,asset_cash,"",0,Банкны төлбөрийн темринал
account_template_1109_0101,11090101,Money on the road,asset_current,"",1,Замд яваа мөнгө
account_template_1110_0101,11100101,Other cash equivalents,asset_current,"",0,Бусад мөнгө түүнтэй адилтгах хөрөнгө
account_template_1201_0101,12010101,Trade receivables Enterprise,asset_receivable,l10n_mn.account_cashflow_tag_111,1,Худалдааны авлага ААН
account_template_1201_0201,12010201,Trade Receivable Individual,asset_receivable,l10n_mn.account_cashflow_tag_111,1,Худалдааны авлага Хувь хүн
account_template_1201_0202,12010202,Trade Receivables Individual (PoS),asset_receivable,l10n_mn.account_cashflow_tag_111,1,Худалдааны авлага Хувь хүн (PoS)
account_template_1201_0301,12010301,Trade Receivables Related Party,asset_receivable,l10n_mn.account_cashflow_tag_111,1,Худалдааны авлага Холбоотой тал
account_template_1202_0101,12020101,Accounts receivable,asset_prepayments,l10n_mn.account_cashflow_tag_123,0,Авлагын бичиг
account_template_1203_0101,12030101,Bad debt deduction,asset_prepayments,l10n_mn.account_cashflow_tag_123,0,Найдваргүй авлагын хасагдуулга
account_template_1204_0101,12040101,Prepaid CIT,asset_prepayments,l10n_mn.account_cashflow_tag_127,0,Урьдчилж төлсөн ААНОАТатвар
account_template_1204_0201,12040201,Prepaid VAT (PRC),asset_prepayments,l10n_mn.account_cashflow_tag_127,0,Урьдчилж төлсөн ХХОАТатвар
account_template_1204_0301,12040301,Prepaid VAT,asset_prepayments,l10n_mn.account_cashflow_tag_127,0,Урьдчилж төлсөн НӨАТатвар
account_template_1204_0302,12040302,Deferred VAT /Buildings/,asset_prepayments,,0,Хойшлуулсан НӨАТатвар /Барилга байгууламж/
account_template_1204_0303,12040303,Deferred VAT /Machines and equipment/,asset_prepayments,,0,Хойшлуулсан НӨАТатвар /Машин тоног төхөөрөмж/
account_template_1204_0304,12040304,Deferred VAT /Exploratory assessment/,asset_prepayments,,0,Хойшлуулсан НӨАТатвар /Хайгуулын үнэлгээ/
account_template_1204_0401,12040401,Customs duty paid in advance,asset_prepayments,l10n_mn.account_cashflow_tag_127,0,Урьдчилж төлсөн Гаалийн албан татвар
account_template_1204_0501,12040501,Prepaid Real Estate Taxes,asset_prepayments,l10n_mn.account_cashflow_tag_127,0,Урьдчилж төлсөн Үл хөдлөх хөрөнгийн албан татвар
account_template_1204_0601,12040601,Prepaid Land Charges,asset_prepayments,l10n_mn.account_cashflow_tag_127,0,Урьдчилж төлсөн Газрын төлбөр
account_template_1204_0701,12040701,Prepaid VAT (АТӨЯХАТ),asset_prepayments,l10n_mn.account_cashflow_tag_127,0,Урьдчилж төлсөн АТӨЯХАТатвар
account_template_1204_0801,12040801,Prepaid Air Pollution Tax,asset_prepayments,l10n_mn.account_cashflow_tag_127,0,Урьдчилж төлсөн Агаарын бохирдлын татвар
account_template_1204_0901,12040901,Prepaid Excise Duty,asset_prepayments,l10n_mn.account_cashflow_tag_127,0,Урьдчилж төлсөн Онцгой албан татвар
account_template_1204_1001,12041001,Prepaid Natural Resource Usage Charges,asset_prepayments,l10n_mn.account_cashflow_tag_127,0,Урьдчилж төлсөн Байгалийн нөөц ашиглалтын төлбөр
account_template_1204_9901,12049901,Other taxes paid in advance,asset_prepayments,l10n_mn.account_cashflow_tag_127,0,Урьдчилж төлсөн Бусад татвар
account_template_1204_9902,12049902,Prepaid Taxes - Current Account,asset_prepayments,l10n_mn.account_cashflow_tag_127,0,Урьдчилж төлсөн татварууд - Харилцах данс
account_template_1205_0101,12050101,"Prepaid ED, NDSH",asset_prepayments,l10n_mn.account_cashflow_tag_122,0,"Урьдчилж төлсөн ЭМ, НДШ"
account_template_1205_0201,12050201,Prepaid Voluntary Insurance Premiums,asset_prepayments,l10n_mn.account_cashflow_tag_128,0,Урьдчилж төлсөн Сайн дурын даатгалын хураамж
account_template_1206_0101,12060101,Payday Advances and Short Term Loans,asset_receivable,l10n_mn.account_cashflow_tag_215_225,1,"Цалингийн урьдчилгаа, богино хугацаат зээл"
account_template_1206_0201,12060201,Payments and shortages,asset_receivable,l10n_mn.account_cashflow_tag_116,1,"Төлбөр, дутагдал"
account_template_1206_0301,12060301,Non-trade receivables Enterprise,asset_receivable,l10n_mn.account_cashflow_tag_116,1,Худалдааны бус авлага ААН
account_template_1206_0401,12060401,Non-trade receivables Individuals,asset_receivable,l10n_mn.account_cashflow_tag_116,1,Худалдааны бус авлага Хувь хүн
account_template_1206_0501,12060501,Non-trade receivables Related party,asset_receivable,l10n_mn.account_cashflow_tag_116,1,Худалдааны бус авлага Холбоотой тал
account_template_1206_0601,12060601,Short-term loan receivables Enterprise,asset_receivable,l10n_mn.account_cashflow_tag_116,1,Богино хугацаат зээлийн авлага ААН
account_template_1206_0701,12060701,Short-term loan receivables Individuals,asset_receivable,l10n_mn.account_cashflow_tag_116,1,Богино хугацаат зээлийн авлага Хувь хүн
account_template_1206_0801,12060801,Short Term Loans Receivable Related Party,asset_receivable,l10n_mn.account_cashflow_tag_116,1,Богино хугацаат зээлийн авлага Холбоотой тал
account_template_1206_0901,12060901,Dividends Receivable,asset_receivable,l10n_mn.account_cashflow_tag_217,1,Ногдол ашгийн авлага
account_template_1206_1001,12061001,Pledged receivables,asset_receivable,l10n_mn.account_cashflow_tag_116,1,Барьцаалсан авлага
account_template_1206_1101,12061101,Interest receivable,asset_receivable,l10n_mn.account_cashflow_tag_116,1,Хүүний авлага
account_template_1206_1201,12061201,Fixed Assets Trade Receivables,asset_receivable,l10n_mn.account_cashflow_tag_211_221,1,Үндсэн хөрөнгө худалдааны авлага
account_template_1206_1301,12061301,Intangible assets trade receivables,asset_receivable,l10n_mn.account_cashflow_tag_212_222,1,Биет бус хөрөнгө худалдааны авлага
account_template_1206_1401,12061401,Investment trade receivables,asset_receivable,l10n_mn.account_cashflow_tag_213_223,1,Хөрөнгө оруулалт худалдааны авлага
account_template_1206_9901,12069901,Other non-trade receivables,asset_receivable,l10n_mn.account_cashflow_tag_116,1,Бусад худалдааны бус авлага
account_template_1301_0101,13010101,Liquid securities,asset_current,,0,Түргэн борлогдох үнэт цаас
account_template_1301_0201,13010201,Term deposits,asset_current,,0,Хугацаат хадгаламж
account_template_1302_0101,13020101,Financial assets at fair value as other comprehensive income,asset_current,,0,Бусад дэлгэрэнгүй орлого болох бодит үнэ цэнээр илэрхийлэх санхүүгийн хөрөнгө
account_template_1303_0101,13030101,Financial cost expressed as amortized cost,asset_current,,0,Хорогдуулсан өртөгөөр илэрхийлэгдэх санхүүгийн өртөг
account_template_1401_0101,14010101,Inventory in warehouse,asset_current,"",0,Агуулахад байгаа бараа материал
account_template_1401_0401,14010401,Difference in download cost,asset_current,"",0,Татан авалтын өртөгийн зөрүү
account_template_1401_0701,14010701,Animal,asset_current,"",0,Амьтан
account_template_1401_0801,14010801,Livestock,asset_current,"",0,Мал
account_template_1401_0901,14010901,Plant,asset_current,"",0,Ургамал
account_template_1402_0101,14020101,Raw materials in storage,asset_current,"",0,Агуулахад байгаа түүхий эд материал
account_template_1403_0301,14030301,The part of fixed assets in the warehouse,asset_current,"",0,Агуулахад байгаа үндсэн хөрөнгийн хэсэг
account_template_1403_0401,14030401,Supplies in stock,asset_current,"",0,Агуулахад байгаа хангамжийн материал
account_template_1403_0501,14030501,Supplies in use,asset_current,"",0,Ашиглалтанд байгаа хангамжийн материал
account_template_1403_0502,14030502,Depreciation of supplies in use,asset_current,"",0,Ашиглалтанд байгаа хангамжийн материалын элэгдэл
account_template_1404_0101,14040101,Work-in-Process - Direct Materials,asset_current,"",0,Дуусаагүй үйлдвэрлэл - Шууд материал
account_template_1404_0201,14040201,Work in progress - Direct labor,asset_current,"",0,Дуусаагүй үйлдвэрлэл - Шууд хөдөлмөр
account_template_1404_0301,14040301,Work in progress - NZ,asset_current,"",0,Дуусаагүй үйлдвэрлэл - НЗ
account_template_1404_0401,14040401,Semi-finished products,asset_current,"",0,Хагас боловсруулсан бүтээгдэхүүн
account_template_1405_0101,14050101,Goods available to the agent,asset_current,"",0,Агентад байгаа бараа
account_template_1406_0101,14060101,Goods on the way,asset_current,"",0,Замд яваа бараа
account_template_1407_0101,14070101,A temporary account for the cost of incoming goods,asset_current,"",0,Ирж буй барааны өртөгийн түр данс
account_template_1408_0101,14080101,Provisional account for the cost of goods issued,asset_current,"",0,Гарч буй барааны өртөгийн түр данс
account_template_1501_0101,15010101,Prepaid rent,asset_prepayments,l10n_mn.account_cashflow_tag_124,0,Урьдчилж төлсөн түрээс
account_template_1501_0201,15010201,Prepaid fees,asset_prepayments,,0,Урьдчилж төлсөн хураамж
account_template_1501_0301,15010301,Prepaid salary,asset_prepayments,,0,Урьдчилж төлсөн цалин
account_template_1501_0901,15010901,Other prepaid expenses,asset_prepayments,l10n_mn.account_cashflow_tag_129,0,Урьдчилж төлсөн бусад зардал
account_template_1502_0101,15020101,Advances paid to preparation suppliers,asset_prepayments,l10n_mn.account_cashflow_tag_123,0,Бэлтгэн нийлүүлэгчид төлсөн урьдчилгаа
account_template_1502_0201,15020201,Advances and assignments of subsequent reports,asset_prepayments,l10n_mn.account_cashflow_tag_124,0,"Дараа тайлангийн урьдчилгаа, томилолт"
account_template_1502_0301,15020301,Advance paid for the purchase of fixed assets,asset_prepayments,l10n_mn.account_cashflow_tag_221,0,Үндсэн хөрөнгө худалдан авахад төлсөн урьдчилгаа
account_template_1502_0401,15020401,Advances paid for the purchase of intangible assets,asset_prepayments,l10n_mn.account_cashflow_tag_222,0,Биет бус хөрөнгө худалдан авахад төлсөн урьдчилгаа
account_template_1502_0501,15020501,Advance paid for investment purchase,asset_prepayments,l10n_mn.account_cashflow_tag_223,0,Хөрөнгө оруулалт худалдан авахад төлсөн урьдчилгаа
account_template_1502_0601,15020601,Advances paid for the purchase of supplies,asset_prepayments,l10n_mn.account_cashflow_tag_123,0,Хангамжийн зүйлс худалдан авахад төлсөн урьдчилгаа
account_template_1502_0901,15020901,Other non-trade advances,asset_prepayments,l10n_mn.account_cashflow_tag_129,0,Бусад худалдааны бус урьдчилгаа
account_template_1601_0101,16010101,Other working capital,asset_current,,0,Бусад эргэлтийн хөрөнгө
account_template_1701_0101,17010101,Non-current assets held for sale,asset_current,,0,Борлуулах зорилгоор эзэмшиж буй эргэлтийн бус хөрөнгө
account_template_2001_0101,20010101,Tangible assets - Land ownership,asset_fixed,,0,Биет хөрөнгө - Газрын сайржуулалт
account_template_2001_0201,20010201,Tangible assets - Depreciation of land improvements,asset_fixed,,0,Биет хөрөнгө - Газрын сайжруулалтын элэгдэл
account_template_2002_0101,20020101,Tangible assets - Buildings and facilities,asset_fixed,,0,"Биет хөрөнгө - Барилга, байгууламж"
account_template_2002_0201,20020201,Tangible assets - Depreciation of buildings and structures,asset_fixed,,0,"Биет хөрөнгө - Барилга, байгууламжын элэгдэл"
account_template_2003_0101,20030101,Tangible assets - machinery and equipment,asset_fixed,,0,Биет хөрөнгө - Машин тоног төхөөрөмж
account_template_2003_0201,20030201,Tangible assets - Depreciation of machinery and equipment,asset_fixed,,0,Биет хөрөнгө - Машин тоног төхөөрөмжийн элэгдэл
account_template_2004_0101,20040101,Tangible assets - Vehicles,asset_fixed,,0,Биет хөрөнгө - Тээврийн хэрэгсэл
account_template_2004_0201,20040201,Tangible assets - Depreciation of vehicles,asset_fixed,,0,Биет хөрөнгө - Тээврийн хэрэгслийн элэгдэл
account_template_2005_0101,20050101,Tangible assets - Furniture and fixtures,asset_fixed,,0,Биет хөрөнгө - Тавилга эд хогшил
account_template_2005_0201,20050201,Tangible assets - Depreciation of furniture,asset_fixed,,0,Биет хөрөнгө - Тавилга эд хогшлын элэгдэл
account_template_2006_0101,20060101,Tangible assets - Computers and peripherals,asset_fixed,,0,"Биет хөрөнгө - Компьютер, дагалдах хэрэгсэл"
account_template_2006_0201,20060201,Tangible assets - Depreciation of computers and peripherals,asset_fixed,,0,"Биет хөрөнгө - Компьютер, дагалдах хэрэгслийн элэгдэл"
account_template_2008_0101,20080101,Tangible assets - Buildings in progress,asset_fixed,,0,Биет хөрөнгө - Дуусаагүй барилга
account_template_2008_0201,20080201,Tangible assets - Depreciation of construction in progress,asset_fixed,,0,Биет хөрөнгө - Дуусаагүй барилгын элэгдэл
account_template_2009_0101,20090101,Tangible assets - Other assets,asset_fixed,,0,Биет хөрөнгө - Бусад хөрөнгө
account_template_2009_0201,20090201,Tangible assets - Depreciation of other assets,asset_fixed,,0,Биет хөрөнгө - Бусад хөрөнгийн элэгдэл
account_template_2101_0101,21010101,Intangible Assets - Copyright,asset_fixed,,0,Биет бус хөрөнгө - Зохиогчийн эрх
account_template_2101_0201,21010201,Intangible Assets - Amortization of Copyright,asset_fixed,,0,Биет бус хөрөнгө - Зохиогчийн эрхийн хорогдол
account_template_2102_0101,21020101,Intangible assets - Software,asset_fixed,,0,Биет бус хөрөнгө - Програм хангамж
account_template_2102_0201,21020201,Intangible assets - Amortization of software,asset_fixed,,0,Биет бус хөрөнгө - Програм хангамжын хорогдол
account_template_2103_0101,21030101,Intangible assets - Patents,asset_fixed,,0,Биет бус хөрөнгө - Патент
account_template_2103_0201,21030201,Intangible Assets - Patent Amortization,asset_fixed,,0,Биет бус хөрөнгө - Патентын хорогдол
account_template_2104_0101,21040101,Intangible assets - Trademarks,asset_fixed,,0,Биет бус хөрөнгө - Барааны тэмдэг
account_template_2104_0201,21040201,Intangible assets - Trademark amortization,asset_fixed,,0,Биет бус хөрөнгө - Барааны тэмдэгийн хорогдол
account_template_2105_0101,21050101,Intangible Assets - Licenses,asset_fixed,,0,Биет бус хөрөнгө - Тусгай зөвшөөрөл
account_template_2105_0201,21050201,Intangible assets - License amortization,asset_fixed,,0,Биет бус хөрөнгө - Тусгай зөвшөөрлийн хорогдол
account_template_2109_0101,21090101,Intangible assets - Other assets,asset_fixed,,0,Биет бус хөрөнгө - Бусад хөрөнгө
account_template_2109_0201,21090201,Intangible assets - Depreciation of other assets,asset_fixed,,0,Биет бус хөрөнгө - Бусад хөрөнгийн элэгдэл
account_template_2401_0101,24010101,Biological assets - Livestock,asset_fixed,,0,Биологийн хөрөнгө - Мал амьтад
account_template_2401_0201,24010201,Biological assets - Loss of livestock,asset_fixed,,0,Биологийн хөрөнгө - Мал амьтадын хорогдол
account_template_2402_0101,24020101,Biological assets - Plants,asset_fixed,,0,Биологийн хөрөнгө - Ургамал
account_template_2402_0201,24020201,Biological assets - Plant mortality,asset_fixed,,0,Биологийн хөрөнгө - Ургамалын хорогдол
account_template_2201_0101,22010101,Financial assets at fair value through profit or loss,asset_non_current,l10n_mn.account_cashflow_tag_211_221,0,Ашиг алдагдлаарх бодит үнэ цэнээр илэрхийлэх санхүүгийн хөрөнгө
account_template_2204_0101,22040101,Investments in subsidiaries and joint ventures,asset_non_current,l10n_mn.account_cashflow_tag_211_221,0,"Хараат компани, хамтарсан үйлдвэрт оруулсан хөрөнгө оруулалт"
account_template_2205_0101,22050101,Investment in Subsidiary,asset_non_current,l10n_mn.account_cashflow_tag_211_221,0,Охин компанид оруулсан хөрөнгө оруулалт
account_template_2301_0101,23010101,Exploration and evaluation assets,asset_non_current,l10n_mn.account_cashflow_tag_211_221,0,Хайгуул ба үнэлгээний хөрөнгө
account_template_2501_0101,25010101,Deferred tax assets,asset_non_current,,0,Хойшлогдсон татварын хөрөнгө
account_template_2601_0101,26010101,Fixed assets for investment purposes,asset_non_current,l10n_mn.account_cashflow_tag_211_221,0,Хөрөнгө оруулалтын зориулалттай үндсэн хөрөнгө
account_template_2601_0201,26010201,Impairment of fixed assets,asset_non_current,,0,Үндсэн хөрөнгийн үнэ цэнийн бууралт
account_template_2701_0101,27010101,Long-term loan receivables Enterprise,asset_non_current,l10n_mn.account_cashflow_tag_211_221,0,Урт хугацаат зээлийн авлага ААН
account_template_2701_0201,27010201,Long-term loan receivables Individuals,asset_non_current,l10n_mn.account_cashflow_tag_211_221,0,Урт хугацаат зээлийн авлага Хувь хүн
account_template_2701_0301,27010301,Long-term loan receivables Related party,asset_non_current,l10n_mn.account_cashflow_tag_211_221,0,Урт хугацаат зээлийн авлага Холбоотой тал
account_template_2709_0101,27090101,Other non-current assets,asset_non_current,,0,Бусад эргэлтийн бус хөрөнгө
account_template_3101_0101,31010101,Accounts Payable to Trade with Enterprise,liability_payable,l10n_mn.account_cashflow_tag_123,1,Худалдааны БН-д өгөх өглөг ААН
account_template_3101_0201,31010201,Accounts Payable to Trade with Individuals,liability_payable,l10n_mn.account_cashflow_tag_123,1,Худалдааны БН-д өгөх өглөг Хувь хүн
account_template_3101_0301,31010301,Accounts Payable to Trade with Related party,liability_payable,l10n_mn.account_cashflow_tag_123,1,Худалдааны БН-д өгөх өглөг Холбоотой тал
account_template_3201_0101,32010101,Prepaid trading income,liability_current,l10n_mn.account_cashflow_tag_123,1,Урьдчилж орсон худалдааны орлого
account_template_3202_0101,32020101,Prepaid rental income,liability_current,l10n_mn.account_cashflow_tag_123,1,Урьдчилж орсон түрээсийн орлого
account_template_3203_0101,32030101,Prepaid interest income,liability_current,l10n_mn.account_cashflow_tag_129,1,Урьдчилж орсон зээлийн хүүгийн орлого
account_template_3301_0101,33010101,Payment of wages,liability_payable,l10n_mn.account_cashflow_tag_121,1,Цалингийн өглөг
account_template_3401_0101,34010101,Tax payable - CIT tax,liability_current,l10n_mn.account_cashflow_tag_127,0,Татварын өглөг - ААНОАТатвар
account_template_3401_0201,34010201,Tax payable - VAT,liability_current,l10n_mn.account_cashflow_tag_127,0,Татварын өглөг - НӨАТатвар
account_template_3401_0301,34010301,Tax payable - VAT (PRC),liability_current,l10n_mn.account_cashflow_tag_127,0,Татварын өглөг - ХХОАТатвар
account_template_3401_0401,34010401,Tax payable - Customs duties,liability_current,l10n_mn.account_cashflow_tag_127,0,Татварын өглөг - Гаалийн албан татвар
account_template_3401_0501,34010501,Tax payable - Real Estate Tax,liability_current,l10n_mn.account_cashflow_tag_127,0,Татварын өглөг - Үл хөдлөх хөрөнгийн албан татвар
account_template_3401_0601,34010601,Tax payable - Land tax,liability_current,l10n_mn.account_cashflow_tag_127,0,Татварын өглөг - Газрын албан татвар
account_template_3401_0701,34010701,Tax payable - VAT (АТӨЯХАТ),liability_current,l10n_mn.account_cashflow_tag_127,0,Татварын өглөг - АТӨЯХАТатвар
account_template_3401_0801,34010801,Tax payable - Air pollution tax,liability_current,l10n_mn.account_cashflow_tag_127,0,Татварын өглөг - Агаарын бохирдлын татвар
account_template_3401_0901,34010901,Tax payable - Excise duty,liability_current,l10n_mn.account_cashflow_tag_127,0,Татварын өглөг - Онцгой албан татвар
account_template_3401_1001,34011001,Tax payable - Natural resource use tax,liability_current,l10n_mn.account_cashflow_tag_127,0,Татварын өглөг - Байгалийн нөөц ашиглалтын татвар
account_template_3401_1101,34011101,Tax payable - NHT,liability_current,l10n_mn.account_cashflow_tag_127,0,Татварын өглөг – НХАТатвар
account_template_3401_9901,34019901,Tax payable - Other taxes,liability_current,l10n_mn.account_cashflow_tag_127,0,Татварын өглөг - Бусад татвар
account_template_3401_9902,34019902,Tax Payable - Current Account,liability_current,l10n_mn.account_cashflow_tag_127,0,Татварын өглөг - Харилцах данс
account_template_3402_0101,34020101,Payment of medical and social insurance,liability_current,l10n_mn.account_cashflow_tag_122,0,"ЭМ, НДШ-ийн өглөг"
account_template_3402_0201,34020201,Voluntary insurance payments,liability_current,l10n_mn.account_cashflow_tag_128,0,Сайн дурын даатгалын өглөг
account_template_3501_0101,35010101,Short-term bank loan payable,liability_payable,l10n_mn.account_cashflow_tag_127,1,Банкны богино хугацаат зээлийн өглөг
account_template_3501_0201,35010201,NBFC short-term loan payable,liability_payable,l10n_mn.account_cashflow_tag_127,1,ББСБ-ын богино хугацаат зээлийн өглөг
account_template_3501_0301,35010301,Payables for personal short-term loans,liability_payable,l10n_mn.account_cashflow_tag_127,1,Хувь хүний богино хугацаат зээлийн өглөг
account_template_3502_0101,35020101,Bank's short-term interest rate,liability_payable,l10n_mn.account_cashflow_tag_127,1,Банкны богино хугацаат зээлийн хүү
account_template_3502_0201,35020201,NBFC's short-term loan interest rate,liability_payable,l10n_mn.account_cashflow_tag_127,1,ББСБ-ын богино хугацаат зээлийн хүү
account_template_3502_0301,35020301,Personal short-term loan interest rates,liability_payable,l10n_mn.account_cashflow_tag_127,1,Хувь хүний богино хугацаат зээлийн хүү
account_template_3601_0101,36010101,Dividend Payable,liability_current,,0,Ногдол ашгийн өглөг
account_template_3701_0101,37010101,Contingent liabilities,liability_current,l10n_mn.account_cashflow_tag_222,0,Нөөц өр төлбөр
account_template_3801_0101,38010101,Insurance payable,liability_payable,l10n_mn.account_cashflow_tag_128,1,Даатгалын өглөг
account_template_3802_0101,38020101,Membership dues,liability_payable,l10n_mn.account_cashflow_tag_129,1,Гишүүнчлэлийн өглөг
account_template_3803_0101,38030101,Payments to employees,liability_payable,l10n_mn.account_cashflow_tag_121,1,Ажилчдад өгөх өглөг
account_template_3804_0101,38040101,Payables to individuals,liability_payable,l10n_mn.account_cashflow_tag_129,1,Хувь хүмүүст өгөх өглөг
account_template_3805_0101,38050101,Guarantees and pledges payable,liability_payable,l10n_mn.account_cashflow_tag_129,1,"Баталгаа, барьцааны өглөг"
account_template_3806_0101,38060101,Payables to non-commercial suppliers - Enterprises,liability_payable,l10n_mn.account_cashflow_tag_129,1,Худалдааны бус бэлтгэн нийлүүлэгчид өгөх өглөг - ААН
account_template_3806_0201,38060201,Payables to non-commercial suppliers - Individuals,liability_payable,l10n_mn.account_cashflow_tag_129,1,Худалдааны бус бэлтгэн нийлүүлэгчид өгөх өглөг - Хувь хүн
account_template_3806_0301,38060301,Payables to non-commercial suppliers - Related parties,liability_payable,l10n_mn.account_cashflow_tag_129,1,Худалдааны бус бэлтгэн нийлүүлэгчид өгөх өглөг - Холбоотой тал
account_template_3806_0401,38060401,Payables for purchased fixed assets,liability_payable,l10n_mn.account_cashflow_tag_128,1,Худалдан авсан үндсэн хөрөнгийн өглөг
account_template_3806_0501,38060501,Accounts Payable for Purchased Investments,liability_payable,l10n_mn.account_cashflow_tag_128,1,Худалдан авсан хөрөнгө оруулалтын өглөг
account_template_3809_0101,38090101,Other short-term liabilities,liability_current,l10n_mn.account_cashflow_tag_222,0,Бусад богино хугацаат өр төлбөр
account_template_3901_0101,39010101,Liabilities related to non-current assets held for sale,liability_current,l10n_mn.account_cashflow_tag_222,0,Борлуулах зорилгоор эзэмшиж буй эргэлтийн бус хөрөнгөнд хамаарах өр
account_template_4001_0101,40010101,Long-term bank loans,liability_non_current,l10n_mn.account_cashflow_tag_212_222,0,Банкны урт хугацаат зээл
account_template_4001_0201,40010201,NBFC Long Term Loan,liability_non_current,l10n_mn.account_cashflow_tag_212_222,0,ББСБ-ын урт хугацаат зээл
account_template_4001_0301,40010301,Long-term loans from individuals,liability_non_current,l10n_mn.account_cashflow_tag_212_222,0,Хувь хүнээс авсан урт хугацаат зээл
account_template_4001_0401,40010401,Long-term related party loans,liability_non_current,l10n_mn.account_cashflow_tag_212_222,0,Холбоотой талын урт хугацаат зээл
account_template_4002_0101,40020101,Long-term reserve liabilities,liability_non_current,l10n_mn.account_cashflow_tag_212_222,0,Урт хугацаат нөөц өр төлбөр
account_template_4003_0101,40030101,Deferred tax debt,liability_non_current,l10n_mn.account_cashflow_tag_212_222,0,Хойшлогдсон татварын өр
account_template_4009_0101,40090101,Other long-term liabilities,liability_non_current,l10n_mn.account_cashflow_tag_212_222,0,Бусад урт хугацаат өр төлбөр
account_template_4101_0101,41010101,Ownership - state,equity,"",0,Эздийн өмч - төрийн
account_template_4102_0101,41020101,Ownership - private,equity,"",0,Эздийн өмч - хувийн
account_template_4103_0101,41030101,Ownership - share,equity,"",0,Эздийн өмч - хувьцаат
account_template_4201_0101,42010101,Pocket stock,equity,"",0,Халаасны хувьцаа
account_template_4301_0101,43010101,Additional paid-in capital,equity,"",0,Нэмж төлөгдсөн капитал
account_template_4401_0101,44010101,Capital revaluation surplus,equity,"",0,Хөрөнгийн дахин үнэлгээний нэмэгдэл
account_template_4501_0101,45010101,Foreign currency translation reserve,equity,"",0,Гадаад валютын хөрвүүлэлтийн нөөц
account_template_4509_0101,45090101,Other parts of the owner's property,equity,"",0,Эздийн өмчийн бусад хэсэг
account_template_4601_0101,46010101,Retained earnings for the reporting period,equity_unaffected,"",0,Тайлант үеийн хуримтлагдсан ашиг
account_template_4601_0201,46010201,Retained earnings from previous periods,equity,"",0,Өмнөх үеийн хуримтлагдсан ашиг
account_template_5101_0101,51010101,Sales revenue,income,l10n_mn.account_cashflow_tag_111,0,Борлуулалтын орлого
account_template_5101_0201,51010201,Sales revenue (VAT exempt),income,l10n_mn.account_cashflow_tag_111,0,Борлуулалтын орлого (НӨАТ-с чөлөөлөгдөх)
account_template_5102_0101,51020101,Rental income,income_other,l10n_mn.account_cashflow_tag_111,0,Түрээсийн орлого
account_template_5103_0101,51030101,Interest income,income_other,l10n_mn.account_cashflow_tag_127,0,Хүүний орлого
account_template_5104_0101,51040101,Dividend income,income_other,l10n_mn.account_cashflow_tag_214_224,0,Ногдол ашгийн орлого
account_template_5105_0101,51050101,Royalty revenue,income_other,l10n_mn.account_cashflow_tag_112,0,Эрхийн шимтгэлийн орлого
account_template_5201_0101,52010101,Count difference gain,income_other,"",0,Тооллогын зөрүүний олз
account_template_5201_0201,52010201,Loss of counting difference,income_other,"",0,Тооллогын зөрүүний алдагдал
account_template_5201_0901,52010901,Other comprehensive income,income_other,l10n_mn.account_cashflow_tag_116,0,Бусад дэлгэрэнгүй орлого
account_template_5202_0101,52020101,Price differences resulting from secondary adjustment of price movements,income_other,,0,Үнэ шилжилтийн хоёрдогч тохируулгын үр дүнд бий болсон үнийн зөрүү
account_template_5203_0101,52030101,"Interest income provided to taxpayers who purchased debt instruments and unit rights for open trading in the primary and secondary markets of foreign and domestic securities",income_other,l10n_mn.account_cashflow_tag_129,0,"Гадаад, дотоодын үнэт цаасны анхдагч болон хоёрдогч зах зээлд нээлттэй арилжаалах өрийн хэрэгсэл, нэгж эрх худалдан авсан албан татвар төлөгчид олгосон хүүгийн орлого"
account_template_5204_0101,52040101,"Income from interest on loans and debt instruments drawn from domestic sources by the Commercial Bank of Mongolia",income_other,l10n_mn.account_cashflow_tag_129,0,"Монгол Улсын арилжааны банкны дотоодын эх үүсвэрээс татсан зээл, өрийн хэрэгсэлд олгосон хүүгийн орлого"
account_template_5205_0101,52050101,Income transferred from the representative office to non-Mongolian taxpayers operating in Mongolia through representative offices,income_other,,0,Төлөөний газраар дамжуулан Монгол Улсад үйл ажиллагаа явуулдаг Монгол Улсад байрладаггүй албан татвар төлөгчид тухайн төлөөний газраас шилжүүлсэн орлого
account_template_5205_0201,52050201,Income related to the activities of the representative office transferred to a person not located in Mongolia,income_other,,0,Төлөөний газрын үйл ажиллагаатай холбогдох орлогыг Монгол Улсад байрладаггүй этгээдэд шилжүүлсэн орлого
account_template_5205_0301,52050301,Profits transferred from the representative office to its parent company in a given tax year,income_other,,0,Тухайн татварын жилд төлөөний газраас өөрийн толгой аж ахуйн нэгжид шилжүүлсэн ашиг
account_template_5205_0401,52050401,Exempt income remitted abroad from own sale of petroleum products,income_other,,0,Газрын тосны бүтээгдэхүүний өөрт ногдох борлуулалтаас гадаадад шилжүүлсэн чөлөөлөгдөх орлого
account_template_5205_0501,52050501,Refundable income under the Environmental Impact Assessment Act,income_other,,0,Байгаль орчинд нөлөөлөх байдлын үнэлгээний тухай хуулийн дагуу буцаан олголтын орлого
account_template_5205_0601,52050601,Income from insurance claims,income_other,l10n_mn.account_cashflow_tag_113,0,Даатгалын нөхөн төлбөрийн орлого
account_template_5301_0101,53010101,Realized foreign exchange gains,income_other,l10n_mn.account_cashflow_tag_400,0,Валютын ханшийн зөрүүний хэрэгжсэн олз
account_template_5301_0201,53010201,Unrealized foreign exchange gains,income_other,l10n_mn.account_cashflow_tag_400,0,Валютын ханшийн зөрүүний хэрэгжээгүй олз
account_template_5302_0101,53020101,Realized loss on exchange rate differences,income_other,l10n_mn.account_cashflow_tag_400,0,Валютын ханшийн зөрүүний хэрэгжсэн гарз
account_template_5302_0201,53020201,Unrealized exchange rate loss,income_other,l10n_mn.account_cashflow_tag_400,0,Валютын ханшийн зөрүүний хэрэгжээгүй гарз
account_template_5401_0101,54010101,Gain on sale of fixed assets,income_other,"",0,Үндсэн хөрөнгө борлуулсны олз
account_template_5401_0201,54010201,Loss on sale of fixed assets,income_other,"",0,Үндсэн хөрөнгө борлуулсны гарз
account_template_5402_0101,54020101,Provisional account for the sale of construction assets,income_other,,0,Барилга байгууламжын хөрөнгө борлуулсны түр данс
account_template_5402_0201,54020201,Provisional account for sale of other fixed assets,income_other,,0,Бусад үндсэн хөрөнгө борлуулсны түр данс
account_template_5501_0101,55010101,Gain on sale of intangible assets,income_other,"",0,Биет бус хөрөнгө борлуулсны олз
account_template_5501_0201,55010201,Loss on sale of intangible assets,income_other,"",0,Биет бус хөрөнгө борлуулсны гарз
account_template_5502_0101,55020101,Provisional account for the sale of intangible assets,income_other,,0,Биет бус хөрөнгө борлуулсны түр данс
account_template_5601_0101,56010101,Gain on sale of investment,income_other,"",0,Хөрөнгө оруулалт борлуулсны олз
account_template_5601_0201,56010201,Loss on sale of investment,income_other,"",0,Хөрөнгө оруулалт борлуулсны гарз
account_template_5701_0101,57010101,Early payment discount gain,income_other,,0,Хугацаанаасаа өмнө төлсний хөнгөлөлтийн олз
account_template_5701_0201,57010201,Early payment discount loss,income_other,,0,Хугацаанаасаа өмнө төлсний хөнгөлөлтийн алдагдал
account_template_5901_0101,59010101,Other gains from non-core activities,income_other,"",0,Үндсэн бус үйл ажиллагааны бусад олз
account_template_5901_0201,59010201,Other losses from non-core activities,income_other,"",0,Үндсэн бус үйл ажиллагааны бусад гарз
account_template_6001_0101,60010101,Sales discount,income,"",0,Борлуулалтын хөнгөлөлт
account_template_6002_0101,60020101,Sales promotion,income,"",0,Борлуулалтын урамшуулал
account_template_6003_0101,60030101,Sales Returns,income,"",0,Борлуулалтын буцаалт
account_template_6004_0101,60040101,Cash discount,income,"",0,Бэлнээр төлсөний хөнгөлөлт
account_template_6005_0101,60050101,Early payment discount,income,"",0,Хугацаанаас өмнө төлсний хөнгөлөлт
account_template_6006_0101,60060101,Reduction in sales price,income,"",0,Борлуулалтын үнийн бууралт
account_template_6101_0101,61010101,Cost of goods sold,expense_direct_cost,"",0,"Борлуулсан бараа, бүтээгдэхүүний өртөг"
account_template_6102_0101,61020101,Cost of services sold,expense_direct_cost,"",0,Борлуулсан үйлчилгээний өртөг
account_template_7001_0101,70010101,Management - Basic Salary Expenses,expense,"",0,Удирд - Үндсэн цалингийн зардал
account_template_7001_0201,70010201,Management - Additional salary costs,expense,"",0,Удирд - Нэмэгдэл цалингийн зардал
account_template_7002_0101,70020101,Management - HIF expenses,expense,"",0,Удирд - ЭМД-ийн зардал
account_template_7002_0201,70020201,Management - Social Security costs,expense,"",0,Удирд - НДШ-ийн зардал
account_template_7002_0301,70020301,Management - Bank fees and charges,expense,"",0,Удирд - Банкны шимтгэл хураамжын зардал
account_template_7002_0401,70020401,Management - Expenses for foreign missions,expense,"",0,Удирд - Гадаад томилолтын зардал
account_template_7002_0402,70020402,Management - Internal Mission Expenses,expense,"",0,Удирд - Дотоод томилолтын зардал
account_template_7002_0501,70020501,Management - Clerical expenses,expense,"",0,Удирд - Бичиг хэргийн зардал
account_template_7002_0601,70020601,Management - Postal costs,expense,"",0,Удирд - Шуудан холбооны зардал
account_template_7002_0602,70020602,Management - Landline costs,expense,"",0,Удирд - Суурин утасны зардал
account_template_7002_0603,70020603,Management - Cell phone costs,expense,"",0,Удирд - Үүрэн утасны зардал
account_template_7002_0604,70020604,Management - Internet costs,expense,"",0,Удирд - Интернетийн зардал
account_template_7002_0701,70020701,Management - Cost of professional services,expense,"",0,Удирд - Мэргэжлийн үйлчилгээний зардал
account_template_7002_0801,70020801,Management - Human Resource Costs,expense,"",0,Удирд - Хүний нөөцийн зардал
account_template_7002_0901,70020901,Management - Newspaper and magazine subscription costs,expense,"",0,Удирд - Сонин сэтгүүл захиалгын зардал
account_template_7002_1001,70021001,Management - Insurance costs,expense,"",0,Удирд - Даатгалын зардал
account_template_7002_1101,70021101,Management - Operating expenses,expense,"",0,Удирд - Ашиглалтын зардал
account_template_7002_1201,70021201,Management - Maintenance costs,expense,"",0,Удирд - Засварын зардал
account_template_7002_1301,70021301,Management - Depreciation expense,expense,"",0,Удирд - Элэгдэл хорогдлын зардал
account_template_7002_1401,70021401,Management - Rental expenses,expense,l10n_mn.account_cashflow_tag_124,0,Удирд - Түрээсийн зардал
account_template_7002_1501,70021501,Management - Security costs,expense,l10n_mn.account_cashflow_tag_124,0,Удирд - Харуул хамгаалалтын зардал
account_template_7002_1601,70021601,Management - Cost of cleaning services,expense,l10n_mn.account_cashflow_tag_124,0,Удирд - Цэвэрлэгээ үйлчилгээний зардал
account_template_7002_1701,70021701,Management - Transportation costs,expense,l10n_mn.account_cashflow_tag_125,0,Удирд - Тээврийн зардал
account_template_7002_1801,70021801,Management - Fuel costs,expense,l10n_mn.account_cashflow_tag_125,0,Удирд - Шатахууны зардал
account_template_7002_1901,70021901,Management - Reception expenses,expense,"",0,Удирд - Хүлээн авалтын зардал
account_template_7002_2001,70022001,Management - Advertising expenses,expense,"",0,Удирд - Зар сурталчилгааны зардал
account_template_7002_2101,70022101,Management - Project development costs,expense,"",0,Удирд - Төсөл хөгжлийн зардал
account_template_7002_2201,70022201,Management - Research and testing costs,expense,"",0,"Удирд - Судалгаа, туршилтын зардал"
account_template_7002_2301,70022301,Management - Shipping and distribution costs,expense,"",0,"Удирд - Хүргэлт, түгээлтийн зардал"
account_template_7002_9901,70029901,Management - Other operating expenses,expense,"",0,Удирд - Үйл ажиллагааны бусад зардал
account_template_7101_0101,71010101,Basic Salary Expense,expense,"",0,Борл - Үндсэн цалингийн зардал
account_template_7101_0201,71010201,Additional salary costs,expense,"",0,Борл - Нэмэгдэл цалингийн зардал
account_template_7102_0101,71020101,HIF costs,expense,"",0,Борл - ЭМД-ийн зардал
account_template_7102_0201,71020201,Cost of Social Security,expense,"",0,Борл - НДШ-ийн зардал
account_template_7102_0301,71020301,Cost of bank commissions,expense,"",0,Борл - Банкны шимтгэл хураамжын зардал
account_template_7102_0401,71020401,Expenditure on foreign missions,expense,"",0,Борл - Гадаад томилолтын зардал
account_template_7102_0402,71020402,Expenses for domestic assignments,expense,"",0,Борл - Дотоод томилолтын зардал
account_template_7102_0501,71020501,Clerical expenses,expense,"",0,Борл - Бичиг хэргийн зардал
account_template_7102_0601,71020601,Postal costs,expense,"",0,Борл - Шуудан холбооны зардал
account_template_7102_0602,71020602,Landline costs,expense,"",0,Борл - Суурин утасны зардал
account_template_7102_0603,71020603,Cell phone costs,expense,"",0,Борл - Үүрэн утасны зардал
account_template_7102_0604,71020604,Internet costs,expense,"",0,Борл - Интернетийн зардал
account_template_7102_0701,71020701,Cost of professional services,expense,"",0,Борл - Мэргэжлийн үйлчилгээний зардал
account_template_7102_0801,71020801,Human Resource Costs,expense,"",0,Борл - Хүний нөөцийн зардал
account_template_7102_0901,71020901,Newspaper subscription costs,expense,"",0,Борл - Сонин сэтгүүл захиалгын зардал
account_template_7102_1001,71021001,Insurance costs,expense,"",0,Борл - Даатгалын зардал
account_template_7102_1101,71021101,Operating expenses,expense,"",0,Борл - Ашиглалтын зардал
account_template_7102_1201,71021201,Repair costs,expense,"",0,Борл - Засварын зардал
account_template_7102_1301,71021301,Depreciation expense,expense,"",0,Борл - Элэгдэл хорогдлын зардал
account_template_7102_1401,71021401,Rental expenses,expense,"",0,Борл - Түрээсийн зардал
account_template_7102_1501,71021501,Cost of guard protection,expense,"",0,Борл - Харуул хамгаалалтын зардал
account_template_7102_1601,71021601,Cost of cleaning services,expense,"",0,Борл - Цэвэрлэгээ үйлчилгээний зардал
account_template_7102_1701,71021701,Shipping costs,expense,l10n_mn.account_cashflow_tag_125,0,Борл - Тээврийн зардал
account_template_7102_1801,71021801,Fuel costs,expense,l10n_mn.account_cashflow_tag_125,0,Борл - Шатахууны зардал
account_template_7102_1901,71021901,Reception expenses,expense,"",0,Борл - Хүлээн авалтын зардал
account_template_7102_2001,71022001,Advertising expenses,expense,"",0,Борл - Зар сурталчилгааны зардал
account_template_7102_2101,71022101,Project development costs,expense,"",0,Борл - Төсөл хөгжлийн зардал
account_template_7102_2201,71022201,Research and testing costs,expense,"",0,"Борл - Судалгаа, туршилтын зардал"
account_template_7102_2301,71022301,Shipping and distribution costs,expense,"",0,"Борл - Хүргэлт, түгээлтийн зардал"
account_template_7102_9901,71029901,Other operating expenses,expense,"",0,Борл - Үйл ажиллагааны бусад зардал
account_template_7201_0101,72010101,Interest expense for working capital financing,expense,l10n_mn.account_cashflow_tag_129,0,Эргэлтийн хөрөнгө санхүүжүүлэх зээлийн хүүгийн зардал
account_template_7202_0101,72020101,Interest expense for financing fixed assets,expense,l10n_mn.account_cashflow_tag_129,0,Үндсэн хөрөнгө санхүүжүүлэх зээлийн хүүгийн зардал
account_template_7203_0101,72030101,Interest costs of project financing loans,expense,l10n_mn.account_cashflow_tag_129,0,Төсөл санхүүжүүлэх зээлийн хүүгийн зардал
account_template_7301_0101,73010101,Exploration and appraisal costs,expense,"",0,"Хайгуул, үнэлгээний зардал"
account_template_7401_0101,74010101,Costs and fines,expense,"",0,"Алданги, торгуулийн зардал"
account_template_7402_0101,74020101,Donations and support costs,expense,"",0,"Хандив, тусламжийн зардал"
account_template_7403_0101,74030101,Bad debt expense,expense,"",0,Найдваргүй авлагын зардал
account_template_7404_0101,74040101,Cost of capital held in contingency fund,expense,"",0,Болзошгүй алдагдлын санд төвлөрүүлсэн хөрөнгийн зардал
account_template_7405_0101,74050101,Franchise costs,expense,"",0,Франчайзын зардал
account_template_7406_0101,74060101,Cost of banking service fees,expense,"",0,Банкны үйлчилгээний шимтгэлийн зардал
account_template_7407_0101,74070101,Asset disposal costs,expense,"",0,Хөрөнгө акталж устгасны зардал
account_template_7408_0101,74080101,Cost of industrial accidents,expense,"",0,Үйлдвэрийн ослын зардал
account_template_7409_0101,74090101,Ancillary production and service costs,expense,"",0,"Туслах үйлдвэрлэл, үйлчилгээний зардал"
account_template_7410_0101,74100101,Impairment cost of fixed assets,expense,"",0,Үндсэн хөрөнгийн үнэ цэнийн бууралтын зардал
account_template_7499_0101,74990101,Other general expenses,expense,"",0,Бусад ерөнхий зардал
account_template_9101_0101,91010101,Income tax expense for the reporting period,expense,"",0,Тайлант үеийн орлогын албан татварын зардал
account_template_9102_0101,91020101,Deferred tax expense,expense,"",0,Хойшлогдсон татварын зардал
account_template_9103_0101,91030101,Other comprehensive non-taxable income,expense,"",0,Татвар ногдохгүй бусад дэлгэрэнгүй орлого
account_template_9301_0101,93010101,The source of the first balance,expense,"",0,Эхний үлдэгдлийн эх үүсвэр
```

## File: data\template\account.fiscal.position-mn.csv

```csv
id,name,auto_apply,country_id,tax_ids/tax_src_id,tax_ids/tax_dest_id,name@mn
fiscal_position_vat0,Domestic Sales - Purchases,1,base.mn,"","",Дотоодын борлуулалт - Худалдан авалт
fiscal_position_vat2,VAT 0% (3000000),"","",account_tax_sale_vat1,account_tax_sale_vat2,НӨАТ 0% (3000000)
"","","","",account_tax_sale_vat4,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat5,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat6,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat7,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat9,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat10,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat11,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat12,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat13,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat14,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat15,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat16,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat17,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat18,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat19,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat20,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat21,account_tax_sale_vat2,""
"","","","",account_tax_sale_vat22,account_tax_sale_vat2,""
fiscal_position_vat3,Without VAT (2000000),"","",account_tax_sale_vat1,account_tax_sale_vat3,НӨАТ-гүй (2000000)
"","","","",account_tax_sale_vat4,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat5,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat6,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat7,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat9,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat10,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat11,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat12,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat13,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat14,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat3,account_tax_sale_vat15,""
"","","","",account_tax_sale_vat16,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat17,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat18,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat19,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat20,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat21,account_tax_sale_vat3,""
"","","","",account_tax_sale_vat22,account_tax_sale_vat3,""
fiscal_position_vat4,Foreign sales - Purchases,1,"",account_tax_sale_vat1,account_tax_sale_vat23,Гадаад борлуулалт - Худалдан авалт
"","","","",account_tax_sale_vat4,account_tax_sale_vat23,""
"","","","",account_tax_sale_vat5,account_tax_sale_vat23,""
"","","","",account_tax_sale_vat6,account_tax_sale_vat23,""
"","","","",account_tax_sale_vat7,account_tax_sale_vat23,""
"","","","",account_tax_sale_vat9,account_tax_sale_vat24,""
"","","","",account_tax_sale_vat10,account_tax_sale_vat24,""
"","","","",account_tax_sale_vat11,account_tax_sale_vat24,""
"","","","",account_tax_sale_vat12,account_tax_sale_vat24,""
"","","","",account_tax_sale_vat13,account_tax_sale_vat24,""
"","","","",account_tax_sale_vat14,account_tax_sale_vat24,""
"","","","",account_tax_sale_vat24,account_tax_sale_vat15,""
"","","","",account_tax_sale_vat16,account_tax_sale_vat24,""
"","","","",account_tax_sale_vat17,account_tax_sale_vat24,""
"","","","",account_tax_sale_vat18,account_tax_sale_vat24,""
"","","","",account_tax_sale_vat19,account_tax_sale_vat24,""
"","","","",account_tax_sale_vat20,account_tax_sale_vat24,""
"","","","",account_tax_sale_vat21,account_tax_sale_vat24,""
"","","","",account_tax_sale_vat22,account_tax_sale_vat24,""
"","","","",account_tax_purchase_vat1,account_tax_purchase_vat2,""

```

## File: data\template\account.tax-mn.csv

```csv
"id","name","amount","price_include","sequence","amount_type","type_tax_use","description","invoice_label","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/tag_ids","repartition_line_ids/document_type","repartition_line_ids/account_id","description@mn"
"account_tax_sale_vat1","10% G","10.0","True","1","percent","sale","VAT goods","10%","account_tax_group1","base","+TT3a[5]","invoice","","НӨАТ бараа"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[5]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_purchase_vat1","10%","10.0","True","1","percent","purchase","Purchase with VAT","10%","account_tax_group4","base","+TT3a[35]||+TT3a[32]","invoice","","НӨАТ-тэй худалдан авалт"
"","","","","","","","","","","tax","","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[35]||-TT3a[32]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_1204_0301",""
"account_tax_purchase_vat2","0%","0.0","True","10","percent","purchase","Purchase without VAT","0%","account_tax_group4","base","+TT3a[32]","invoice","","НӨАТ-гүй худалдан авалт"
"","","","","","","","","","","tax","","invoice","",""
"","","","","","","","","","","base","-TT3a[32]","refund","",""
"","","","","","","","","","","tax","","refund","",""
"account_tax_sale_vat2","0%","0.0","True","10","percent","sale","VAT 0% sales","0%","account_tax_group6","base","+TT3a[2]","invoice","","НӨАТ 0% борлуулалт"
"","","","","","","","","","","tax","","invoice","",""
"","","","","","","","","","","base","-TT3a[2]","refund","",""
"","","","","","","","","","","tax","","refund","",""
"account_tax_sale_vat3","0% EXEMPT","0.0","True","10","percent","sale","VAT exempt sales","0%","account_tax_group6","base","+TT3a[2]","invoice","","НӨАТ чөлөөлөх борлуулалт"
"","","","","","","","","","","tax","","invoice","",""
"","","","","","","","","","","base","-TT3a[2]","refund","",""
"","","","","","","","","","","tax","","refund","",""
"account_tax_sale_vat4","10% O G","10.0","True","20","percent","sale","VAT sales of other goods","10%","account_tax_group1","base","+TT3a[6]","invoice","","НӨАТ бусад барааны борлуулалт"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[6]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat5","10%","10.0","True","20","percent","sale","VAT sales","10%","account_tax_group1","base","+TT3a[7]","invoice","","НӨАТ эрх борлуулалт"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[7]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat6","10% Liq","10.0","True","20","percent","sale","Sales during VAT liquidation","10%","account_tax_group1","base","+TT3a[8]","invoice","","НӨАТ татан буугдах үеийн борлуулалт"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[8]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat7","10% G D","10.0","True","20","percent","sale","Goods given in payment of VAT debt","10%","account_tax_group1","base","+TT3a[9]","invoice","","НӨАТ өрийн төлбөрт өгсөн бараа"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[9]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat9","10% S D","10.0","True","20","percent","sale","Services provided for payment of VAT debts","10%","account_tax_group2","base","+TT3a[11]","invoice","","НӨАТ өрийн төлбөрт өгсөн үйлчилгээ"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[11]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat10","10% E","10.0","True","20","percent","sale","VAT energy (electricity, heat, water, post, communications)","10%","account_tax_group2","base","+TT3a[12]","invoice","","НӨАТ цахилгаан, дулаан, ус, шуудан, холбоо"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[12]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat11","10% G R","10.0","True","20","percent","sale","Renting and owning VAT goods","10%","account_tax_group2","base","+TT3a[13]","invoice","","НӨАТ бараа түрээслүүлэх, эзэмшүүлэх"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[13]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat12","10% P R","10.0","True","20","percent","sale","Renting and owning VAT premises","10%","account_tax_group2","base","+TT3a[14]","invoice","","НӨАТ байр түрээслүүлэх, эзэмшүүлэх"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[14]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat13","10% L O","10.0","True","20","percent","sale","Leasing and ownership of VAT assets","10%","account_tax_group2","base","+TT3a[15]","invoice","","НӨАТ хөрөнгө түрээслүүлэх, эзэмшүүлэх"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[15]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat14","10% W","10.0","True","20","percent","sale","VAT works, designs and copyrights","10%","account_tax_group2","base","+TT3a[16]","invoice","","НӨАТ бүтээл, загвар, зохиогчийн эрх"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[16]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat15","10% Lot","10.0","True","20","percent","sale","VAT Lotteries, Riddles and Bets","10%","account_tax_group2","base","+TT3a[17]","invoice","","НӨАТ хонжворт сугалаа, таавар, бооцоо"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[17]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat16","10% Med","10.0","True","20","percent","sale","VAT mediation services","10%","account_tax_group2","base","+TT3a[18]","invoice","","НӨАТ зуучлалын үйлчилгээ"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[18]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat17","10% I","10.0","True","20","percent","sale","Interest, penalties and interest on VAT","10%","account_tax_group2","base","+TT3a[19]","invoice","","НӨАТ авсан хүү торгууль, алданги"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[19]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat18","10% V S","10.0","True","20","percent","sale","VAT Asset Valuation Services","10%","account_tax_group2","base","+TT3a[20]","invoice","","НӨАТ хөрөнгийн үнэлгээний үйлчилгээ"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[20]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat19","10% B","10.0","True","20","percent","sale","VAT budget financing and subsidies","10%","account_tax_group2","base","+TT3a[21]","invoice","","НӨАТ төсвийн санхүүжилт, татаас"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[21]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat20","10% L S","10.0","True","20","percent","sale","VAT legal services","10%","account_tax_group2","base","+TT3a[22]","invoice","","НӨАТ хууль зүйн үйлчилгээ"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[22]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat21","10% Hair","10.0","True","20","percent","sale","VAT hairdressing","10%","account_tax_group2","base","+TT3a[23]","invoice","","НӨАТ үсчин, гоо сайхан, угаалга, цэвэрлэгээ"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[23]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat22","10% O S","10.0","True","20","percent","sale","VAT other services","10%","account_tax_group2","base","+TT3a[24]","invoice","","НӨАТ бусад үйлчилгээ"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[24]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat23","0% EX G","0.0","True","30","percent","sale","VAT 0% export goods","0%","account_tax_group3","base","+TT3a[27]","invoice","","НӨАТ 0% экспорт бараа"
"","","","","","","","","","","tax","","invoice","",""
"","","","","","","","","","","base","-TT3a[27]","refund","",""
"","","","","","","","","","","tax","","refund","",""
"account_tax_sale_vat24","0% EX S","0.0","True","30","percent","sale","VAT 0% export service","0%","account_tax_group3","base","+TT3a[28]","invoice","","НӨАТ 0% экспорт үйлчилгээ"
"","","","","","","","","","","tax","","invoice","",""
"","","","","","","","","","","base","-TT3a[28]","refund","",""
"","","","","","","","","","","tax","","refund","",""
"account_tax_purchase_vat3","10% EX","10.0","True","40","percent","purchase","VAT on imported goods and services","10%","account_tax_group4","base","+TT3a[32]||+TT3a[34]","invoice","","НӨАТ импортоор авсан бараа, үйлчилгээ"
"","","","","","","","","","","tax","","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[32]||-TT3a[34]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_1204_0301",""
"account_tax_purchase_vat4","10% T R","10.0","True","40","percent","purchase","Purchased at the time of VAT registration","10%","account_tax_group4","base","+TT3a[32]||+TT3a[36]","invoice","","НӨАТ төлөгчөөр бүртгүүлэх үед худалдан авсан"
"","","","","","","","","","","tax","","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[32]||-TT3a[36]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_1204_0301",""
"account_tax_purchase_vat5","10% L F","10.0","True","40","percent","purchase","VAT was purchased from a livestock farmer","10%","account_tax_group4","base","+TT3a[32]||+TT3a[41]","invoice","","НӨАТ мал аж ахуй эрхлэгчээс худалдан авсан"
"","","","","","","","","","","tax","","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[32]||-TT3a[41]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_1204_0301",""
"account_tax_purchase_vat6","10% Car","10.0","True","40","percent","purchase","VAT purchase of cars and parts (domestic)","10%","account_tax_group4","base","+TT3a[32]||+TT3a[35]","invoice","","НӨАТ автомашин, эд анги худалдан авалт (дотоодын)"
"","","","","","","","","","","tax","+TT3a[44]","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[32]||-TT3a[35]","refund","",""
"","","","","","","","","","","tax","-TT3a[44]","refund","account_template_1204_0301",""
"account_tax_purchase_vat7","10% G","10.0","True","40","percent","purchase","Goods purchased for VAT purposes (domestic)","10%","account_tax_group4","base","+TT3a[32]||+TT3a[35]","invoice","","НӨАТ ажлын хэрэгцээнд авсан бараа (дотоодын)"
"","","","","","","","","","","tax","+TT3a[45]","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[32]||-TT3a[35]","refund","",""
"","","","","","","","","","","tax","-TT3a[45]","refund","account_template_1204_0301",""
"account_tax_purchase_vat8","10% EX MACH","10.0","True","40","percent","purchase","VAT on imported machinery","10%","account_tax_group4","base","+TT3a[32]||+TT3a[38]","invoice","","НӨАТ Машин, тоног төхөөрөмж бэлтгэхэд зориулж импортоор худалдан авсан"
"","","","","","","","","","","tax","","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[32]||-TT3a[38]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_1204_0301",""
"account_tax_purchase_vat9","10% EXEMPT","10.0","True","40","percent","purchase","Goods and services for VAT-exempt production (domestic)","10%","account_tax_group4","base","+TT3a[32]||+TT3a[35]","invoice","","НӨАТ чөлөөлөгдөх үйлдвэрлэлд зориулсан бараа, үйлчилгээ (дотоодын)"
"","","","","","","","","","","tax","+TT3a[46]","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[32]||-TT3a[35]","refund","",""
"","","","","","","","","","","tax","-TT3a[46]","refund","account_template_1204_0301",""
"account_tax_purchase_vat10","10% EX E","10.0","True","40","percent","purchase","VAT Imported goods and services for exploration and preparation assets","10%","account_tax_group4","base","+TT3a[32]","invoice","","НӨАТ хайгуулын үнэлгээний хөрөнгө бэлтгэхэд зориулсан импортын бараа, үйлчилгээ"
"","","","","","","","","","","tax","+TT3a[47]","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[32]","refund","",""
"","","","","","","","","","","tax","-TT3a[47]","refund","account_template_1204_0301",""
"account_tax_purchase_vat11","10% EX NV","10.0","True","40","percent","purchase","Other non-VAT imported goods and services (domestic)","10%","account_tax_group4","base","+TT3a[32]||+TT3a[35]","invoice","","НӨАТ импортын бусад хамааралгүй бараа, үйлчилгээ (дотоодын)"
"","","","","","","","","","","tax","+TT3a[48]","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[32]||-TT3a[35]","refund","",""
"","","","","","","","","","","tax","-TT3a[48]","refund","account_template_1204_0301",""
"account_tax_sale_vat25","10% D F L","10.0","True","50","percent","sale","VAT domestically sold finance lease","10%","account_tax_group5","base","+TT3a[50]","invoice","","НӨАТ дотоодод зарсан санхүүгийн түрээс"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[50]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_sale_vat26","10% Fact S","10.0","True","50","percent","sale","VAT factoring and forfeiting services","10%","account_tax_group5","base","+TT3a[51]","invoice","","НӨАТ Факторинг, форфайтинг хэлцлийн үйлчилгээ"
"","","","","","","","","","","tax","","invoice","account_template_3401_0201",""
"","","","","","","","","","","base","-TT3a[51]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_3401_0201",""
"account_tax_purchase_vat12","10% F L","10.0","True","50","percent","purchase","VAT Finance Lease Purchase","10%","account_tax_group5","base","+TT3a[32]||+TT3a[53]","invoice","","НӨАТ Санхүүгийн түрээсийн худалдан авалт"
"","","","","","","","","","","tax","","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[32]||-TT3a[53]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_1204_0301",""
"account_tax_purchase_vat13","10% Fact","10.0","True","50","percent","purchase","VAT factoring and forfeiting purchases","10%","account_tax_group5","base","+TT3a[32]||+TT3a[54]","invoice","","НӨАТ Факторинг, форфайтинг худалдан авалт"
"","","","","","","","","","","tax","","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[32]||-TT3a[54]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_1204_0301",""
"account_tax_purchase_vat14","VAT Import for preparation of construction assets","10.0","True","50","percent","purchase","","10,00%","account_tax_group4","base","+TT3a[32]||+TT3a[37]","invoice","","НӨАТ Барилга байгууламж бэлтгэхэд зориулж импортоор худалдан авсан"
"","","","","","","","","","","tax","","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[32]||-TT3a[37]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_1204_0301",""
"account_tax_purchase_vat15","VAT Import for preparation of other fixed assets","10.0","True","50","percent","purchase","","10,00%","account_tax_group4","base","+TT3a[32]||+TT3a[39]","invoice","","НӨАТ Бусад үндсэн хөрөнгө бэлтгэхэд зориулж импортоор худалдан авсан"
"","","","","","","","","","","tax","","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[32]||-TT3a[39]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_1204_0301",""
"account_tax_purchase_vat16","VAT Import for operational purposes","10.0","True","50","percent","purchase","","10,00%","account_tax_group4","base","+TT3a[32]||+TT3a[40]","invoice","","НӨАТ Ашиглалтын ажиллагаанд зориулж импортоор худалдан авсан"
"","","","","","","","","","","tax","","invoice","account_template_1204_0301",""
"","","","","","","","","","","base","-TT3a[32]||-TT3a[40]","refund","",""
"","","","","","","","","","","tax","","refund","account_template_1204_0301",""

```

## File: data\template\account.tax.group-mn.csv

```csv
id,name,sequence,country_id,"tax_payable_account_id","tax_receivable_account_id",name@mn
account_tax_group1,Goods subject to VAT,10,base.mn,"account_template_3401_9902","account_template_1204_9902",НӨАТ ногдох бараа
account_tax_group2,Services subject to VAT,10,base.mn,"account_template_3401_9902","account_template_1204_9902",НӨАТ ногдох үйлчилгээ
account_tax_group3,Export sales,11,base.mn,"account_template_3401_9902","account_template_1204_9902",Экспортын борлуулалт
account_tax_group4,Purchased goods and services,12,base.mn,"account_template_3401_9902","account_template_1204_9902","Худалдан авсан бараа, үйлчилгээ"
account_tax_group5,Finance lease item,13,base.mn,"account_template_3401_9902","account_template_1204_9902",Санхүүгийн түрээсийн зүйл
account_tax_group6,Exemption from VAT,14,base.mn,"account_template_3401_9902","account_template_1204_9902",НӨАТ-с чөлөөлөгдөх

```

## File: models\template_mn.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('mn')
    def _get_mn_template_data(self):
        return {
            'property_account_receivable_id': 'account_template_1201_0201',
            'property_account_payable_id': 'account_template_3101_0201',
            'property_account_expense_categ_id': 'account_template_6101_0101',
            'property_account_income_categ_id': 'account_template_5101_0101',
            'property_stock_account_input_categ_id': 'account_template_1407_0101',
            'property_stock_account_output_categ_id': 'account_template_1408_0101',
            'property_stock_valuation_account_id': 'account_template_1401_0101',
            'code_digits': '8',
        }

    @template('mn', 'res.company')
    def _get_mn_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.mn',
                'bank_account_code_prefix': '11',
                'cash_account_code_prefix': '10',
                'transfer_account_code_prefix': '1109',
                'account_default_pos_receivable_account_id': 'account_template_1201_0202',
                'income_currency_exchange_account_id': 'account_template_5301_0201',
                'expense_currency_exchange_account_id': 'account_template_5302_0201',
                'account_journal_early_pay_discount_loss_account_id': 'account_template_5701_0201',
                'account_journal_early_pay_discount_gain_account_id': 'account_template_5701_0101',
                'account_sale_tax_id': 'account_tax_sale_vat1',
                'account_purchase_tax_id': 'account_tax_purchase_vat1',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_mn

```

