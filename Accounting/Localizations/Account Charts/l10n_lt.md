# Odoo Module: l10n_lt

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
    'name': 'Lithuania - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['lt'],
    'version': '1.0.0',
    'description': """
Chart of Accounts (COA) Template for Lithuania's Accounting.

This module also includes:

* List of available banks in Lithuania.
* Tax groups.
* Most common Lithuanian Taxes.
* Fiscal positions.
* Account Tags.
    """,
    'license': 'LGPL-3',
    'author': 'Focusate',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'category': 'Accounting/Localizations/Account Charts',
    'depends': [
        'account',
    ],
    'auto_install': ['account'],
    'data': [
        'data/account_account_tag_data.xml',
        'data/res_bank_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'installable': True,
}

```

## File: data\account_account_tag_data.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo noupdate="1">
  <record id="account_account_tag_a_1_1" model="account.account.tag">
    <field name="name">A.1.1. Assets arising from development</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_1_2" model="account.account.tag">
    <field name="name">A.1.2. Goodwill</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_1_3" model="account.account.tag">
    <field name="name">A.1.3. Software</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_1_4" model="account.account.tag">
    <field name="name">A.1.4. Concessions, patents, licenses, trade marks and similar rights</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_1_5" model="account.account.tag">
    <field name="name">A.1.5. Other intangible assets</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_1_6" model="account.account.tag">
    <field name="name">A.1.6. Advance payments</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_1" model="account.account.tag">
    <field name="name">A.2.1. Land</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_2" model="account.account.tag">
    <field name="name">A.2.2. Buildings and structures</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_3" model="account.account.tag">
    <field name="name">A.2.3. Machinery and plant</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_4" model="account.account.tag">
    <field name="name">A.2.4. Vehicles</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_5" model="account.account.tag">
    <field name="name">A.2.5. Other equipment, fittings and tools</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_6_1" model="account.account.tag">
    <field name="name">A.2.6.1. Land</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_6_2" model="account.account.tag">
    <field name="name">A.2.6.2. Buildings</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_2_7" model="account.account.tag">
    <field name="name">A.2.7. Advance payments and tangible assets under construction (production)</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_1" model="account.account.tag">
    <field name="name">A.3.1. Shares in entities of the entities group</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_2" model="account.account.tag">
    <field name="name">A.3.2. Loans to entities of the entities group</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_3" model="account.account.tag">
    <field name="name">A.3.3. Amounts receivable from entities of the entities group</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_4" model="account.account.tag">
    <field name="name">A.3.4. Shares in associated entities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_5" model="account.account.tag">
    <field name="name">A.3.5. Loans to associated entities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_6" model="account.account.tag">
    <field name="name">A.3.6. Amounts receivable from the associated entities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_7" model="account.account.tag">
    <field name="name">A.3.7. Long-term investments</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_8" model="account.account.tag">
    <field name="name">A.3.8. Amounts receivable after one year</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_3_9" model="account.account.tag">
    <field name="name">A.3.9. Other financial assets</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_4_1" model="account.account.tag">
    <field name="name">A.4.1. Assets of the deferred tax on profit</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_4_2" model="account.account.tag">
    <field name="name">A.4.2. Biological assets</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_a_4_3" model="account.account.tag">
    <field name="name">A.4.3. Other assets</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_1_1" model="account.account.tag">
    <field name="name">B.1.1. Raw materials, materials and consumables</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_1_2" model="account.account.tag">
    <field name="name">B.1.2. Production and work in progress</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_1_3" model="account.account.tag">
    <field name="name">B.1.3. Finished goods</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_1_4" model="account.account.tag">
    <field name="name">B.1.4. Goods for resale</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_1_5" model="account.account.tag">
    <field name="name">B.1.5. Biological assets</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_1_6" model="account.account.tag">
    <field name="name">B.1.6. Fixed tangible assets held for sale</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_1_7" model="account.account.tag">
    <field name="name">B.1.7. Advance payments</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_2_1" model="account.account.tag">
    <field name="name">B.2.1. Trade debtors</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_2_2" model="account.account.tag">
    <field name="name">B.2.2. Amounts owed by entities of the entities group</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_2_3" model="account.account.tag">
    <field name="name">B.2.3. Amounts owed by associates entities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_2_4" model="account.account.tag">
    <field name="name">B.2.4. Other debtors</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_3_1" model="account.account.tag">
    <field name="name">B.3.1. Shares in entities of the entities group</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_3_2" model="account.account.tag">
    <field name="name">B.3.2. Other investments</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_b_4" model="account.account.tag">
    <field name="name">B.4. CASH AND CASH EQUIVALENTS</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_c_prepayments_accrued_income" model="account.account.tag">
    <field name="name">C. PREPAYMENTS AND ACCRUED INCOME</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_1_1" model="account.account.tag">
    <field name="name">D.1.1. Authorized (subscribed) or primary capital</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_1_2" model="account.account.tag">
    <field name="name">D.1.2. Subscribed capital unpaid (–)</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_1_3" model="account.account.tag">
    <field name="name">D.1.3. Own shares (–)</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_2_share_premium" model="account.account.tag">
    <field name="name">D.2. SHARE PREMIUM ACCOUNT</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_3_revaluation_reserve" model="account.account.tag">
    <field name="name">D.3. REVALUATION RESERVE</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_4_1" model="account.account.tag">
    <field name="name">D.4.1. Compulsory reserve or emergency (reserve) capital</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_4_2" model="account.account.tag">
    <field name="name">D.4.2. Reserve for acquiring own shares</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_4_3" model="account.account.tag">
    <field name="name">D.4.3. Other reserves</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_5_1" model="account.account.tag">
    <field name="name">D.5.1. Profit (loss) for the reporting year </field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_5_2" model="account.account.tag">
    <field name="name">D.5.2. Profit (loss) brought forward</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_d_6" model="account.account.tag">
    <field name="name">D.6. Common Summary of Accounts</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_e_grants_subsidies" model="account.account.tag">
    <field name="name">E. GRANTS, SUBSIDIES</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_f_1" model="account.account.tag">
    <field name="name">F.1. Provisions for pensions and similar obligations</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_f_2" model="account.account.tag">
    <field name="name">F.2. Provisions for taxation</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_f_3" model="account.account.tag">
    <field name="name">F.3. Other provisions</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_1" model="account.account.tag">
    <field name="name">G.1.1. Debenture loans</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_2" model="account.account.tag">
    <field name="name">G.1.2. Amounts owed to credit institutions</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_3" model="account.account.tag">
    <field name="name">G.1.3. Payments received on account</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_4" model="account.account.tag">
    <field name="name">G.1.4. Trade creditors</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_5" model="account.account.tag">
    <field name="name">G.1.5. Amounts payable under the bills and checks </field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_6" model="account.account.tag">
    <field name="name">G.1.6. Amounts payable to the entities of the entities group</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_7" model="account.account.tag">
    <field name="name">G.1.7. Amounts payable to the associated entities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_1_8" model="account.account.tag">
    <field name="name">G.1.8. Other amounts payable and long-term liabilities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_1" model="account.account.tag">
    <field name="name">G.2.1. Debenture loans</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_2" model="account.account.tag">
    <field name="name">G.2.2. Amounts owed to credit institutions</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_3" model="account.account.tag">
    <field name="name">G.2.3. Payments received on account</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_4" model="account.account.tag">
    <field name="name">G.2.4. Trade creditors</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_5" model="account.account.tag">
    <field name="name">G.2.5. Amounts payable under the bills and checks </field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_6" model="account.account.tag">
    <field name="name">G.2.6. Amounts payable to the entities of the entities group</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_7" model="account.account.tag">
    <field name="name">G.2.7. Amounts payable to the associated entities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_8" model="account.account.tag">
    <field name="name">G.2.8. Liabilities of tax on profit</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_9" model="account.account.tag">
    <field name="name">G.2.9. Liabilities related to employment relations</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_g_2_10" model="account.account.tag">
    <field name="name">G.2.10. Other amounts payable and short-term liabilities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_h_accruals_deferred_income" model="account.account.tag">
    <field name="name">H. ACCRUALS AND DEFERRED INCOME</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_1_net_turnover" model="account.account.tag">
    <field name="name">1. Net turnover</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_2_cost_of_sales" model="account.account.tag">
    <field name="name">2. Cost of sales</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_3_adjustments_of_biological_assets" model="account.account.tag">
    <field name="name">3. Fair value adjustments of the biological assets</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_4_selling_expenses" model="account.account.tag">
    <field name="name">4. Selling expenses</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_5_general_administrative_expenses" model="account.account.tag">
    <field name="name">5. General and administrative expenses</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_6_other_operating_results" model="account.account.tag">
    <field name="name">6. Other operating results</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_7_income_investments_parent" model="account.account.tag">
    <field name="name">7. Income from investments in the shares of parent, subsidiaries and associated entities</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_8_income_other_longterm_investments_loans" model="account.account.tag">
    <field name="name">8. Income from other long-term investments and loans</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_9_other_interest_similar_income" model="account.account.tag">
    <field name="name">9. Other interest and similar income</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_10_impaired_fin_assets_short_investments" model="account.account.tag">
    <field name="name">10. The impairment of the financial assets and short-term investments</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_11_interest_other_similar_expenses" model="account.account.tag">
    <field name="name">11. Interest and other similar expenses</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
  <record id="account_account_tag_12_tax_on_profit" model="account.account.tag">
    <field name="name">12. Tax on profit</field>
    <field name="applicability">accounts</field>
    <field name="color" eval="8"/>
  </record>
</odoo>

```

## File: data\res_bank_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo noupdate="1">
    <record id="res_bank_siauliu" model="res.bank">
        <field name="name">AB Šiaulių bankas</field>
        <field name="bic">CBSBLT26</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Šiauliai</field>
        <field name="street">Tilžės g.149</field>
        <field name="zip">LT-76348</field>
        <field name="phone">+370 4 1595607</field>
        <field name="email">info@sb.lt</field>
        <field name="active" eval="1"/>
    </record>
    <record id="res_bank_citadele" model="res.bank">
        <field name="name">AB "Citadele" bankas</field>
        <field name="bic">INDULT2X</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">K. Kalinausko g. 13</field>
        <field name="zip">LT-03107</field>
        <field name="phone">+370 5 2664600</field>
        <field name="email">info@citadele.lt</field>
        <field name="active" eval="1"/>
    </record>
    <record id="res_bank_danske" model="res.bank">
        <field name="name">Danske bank A/S Lietuvos filialas</field>
        <field name="bic">SMPOLT22</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">Saltoniškių g. 2</field>
        <field name="zip">LT-08500</field>
        <field name="phone">+370 5 215666</field>
        <field name="email">info@danskebank.lt</field>
        <field name="active" eval="1"/>
    </record>
    <record id="res_bank_luminor" model="res.bank">
        <field name="name">Luminor Bank AB</field>
        <field name="bic">AGBLLT2X</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">Konstitucijos pr. 21A</field>
        <field name="zip">LT-08130</field>
        <field name="phone">+370 5 2393444</field>
        <field name="email">info@luminor.lt</field>
        <field name="active" eval="1"/>
    </record>
    <record id="res_bank_paysera" model="res.bank">
        <field name="name">UAB "Paysera LT"</field>
        <field name="bic">EVIULT21</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">Mėnulio g. 7</field>
        <field name="zip">LT-04326</field>
        <field name="phone">+370 5 2071558</field>
        <field name="email">info@paysera.lt</field>
        <field name="active" eval="1"/>
    </record>
    <record id="res_bank_seb" model="res.bank">
        <field name="name">AB SEB bankas</field>
        <field name="bic">CBVILT2X</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">Gedimino pr. 12,</field>
        <field name="zip">LT-01103</field>
        <field name="phone">+370 5 2682800</field>
        <field name="email">info@seb.lt</field>
        <field name="active" eval="1"/>
    </record>
    <record id="res_bank_swedbank" model="res.bank">
        <field name="name">"Swedbank", AB</field>
        <field name="bic">HABALT22</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">Konstitucijos pr. 20A</field>
        <field name="zip">LT-03502</field>
        <field name="phone">+370 5 2684444</field>
        <field name="email">info@swedbak.lt</field>
        <field name="active" eval="1"/>
    </record>
    <record id="res_bank_medicinos" model="res.bank">
        <field name="name">UAB Medicinos bankas</field>
        <field name="bic">MDBALT22</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">Pamėnkalnio g. 40</field>
        <field name="zip">LT-01114</field>
        <field name="phone">+370 800 60700</field>
        <field name="email">info@medbank.lt</field>
    </record>
    <record id="res_bank_nordea" model="res.bank">
        <field name="name">Nordea Bank AB</field>
        <field name="bic">NDEALT2X</field>
        <field name="country" ref="base.lt"/>
        <field name="city">Vilnius</field>
        <field name="street">Didžioji g. 18</field>
        <field name="zip">LT-01128</field>
        <field name="phone">+370 5 2361361</field>
        <field name="email">info@nordea.lt</field>
        <field name="active" eval="1"/>
    </record>
</odoo>

```

## File: data\template\account.account-lt.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@lt"
"account_account_template_1130","Software Acquisition Cost","1130","asset_non_current","l10n_lt.account_account_tag_a_1_3","False","Programinės įrangos įsigijimo savikaina"
"account_account_template_1138","Amortization of Software Value (-)","1138","asset_non_current","l10n_lt.account_account_tag_a_1_3","False","Programinės įrangos vertės amortizacija (−)"
"account_account_template_1200","Land Acquisition Cost","1200","asset_fixed","l10n_lt.account_account_tag_a_2_1","False","Žemės įsigijimo savikaina"
"account_account_template_1201","Change of Land Value after Revaluation","1201","asset_fixed","l10n_lt.account_account_tag_a_2_1","False","Žemės vertės pokytis dėl perkainojimo"
"account_account_template_1210","Acquisition Cost of Buildings","1210","asset_fixed","l10n_lt.account_account_tag_a_2_2","False","Pastatų ir statinių įsigijimo savikaina"
"account_account_template_1212","Buildings being Prepared for Use","1212","asset_fixed","l10n_lt.account_account_tag_a_2_2","False","Ruošiami naudoti pastatai ir statiniai"
"account_account_template_1217","Depreciation of the Acquisition Cost of Buildings (-)","1217","asset_fixed","l10n_lt.account_account_tag_a_2_2","False","Pastatų ir statinių įsigijimo savikainos nusidėvėjimas (−)"
"account_account_template_1220","Acquisition Cost of Machinery and Equipment","1220","asset_fixed","l10n_lt.account_account_tag_a_2_3","False","Mašinų ir įrangos įsigijimo savikaina"
"account_account_template_1222","Machinery and Equipment being Prepared for Use","1222","asset_fixed","l10n_lt.account_account_tag_a_2_3","False","Ruošiamos naudoti mašinos ir įranga"
"account_account_template_1227","Depreciation of the Acquisition Cost of Machinery and Equipment (-)","1227","asset_fixed","l10n_lt.account_account_tag_a_2_3","False","Mašinų ir įrangos įsigijimo savikainos nusidėvėjimas (−)"
"account_account_template_1230","Acquisition Cost of Vehicles","1230","asset_fixed","l10n_lt.account_account_tag_a_2_4","False","Transporto priemonių įsigijimo savikaina"
"account_account_template_1232","Vehicles being Prepared for Use","1232","asset_fixed","l10n_lt.account_account_tag_a_2_4","False","Ruošiamos naudoti transporto priemonės"
"account_account_template_1237","Depreciation of the Acquisition Cost of Vehicles (-)","1237","asset_fixed","l10n_lt.account_account_tag_a_2_4","False","Transporto priemonių įsigijimo savikainos nusidėvėjimas (−)"
"account_account_template_1240","Acquisition Cost of Other Equipment, Appliances and Tools","1240","asset_fixed","l10n_lt.account_account_tag_a_2_5","False","Kitų įrenginių, prietaisų ir įrankių įsigijimo savikaina"
"account_account_template_1242","Other Equipment, Appliances and Tools being Prepared for Use","1242","asset_fixed","l10n_lt.account_account_tag_a_2_5","False","Ruošiami naudoti kiti įrenginiai, prietaisai ir įrankiai"
"account_account_template_1247","Depreciation of the Acquisition Cost of Other Equipment, Appliances and Tools (-)","1247","asset_fixed","l10n_lt.account_account_tag_a_2_5","False","Kitų įrenginių, prietaisų ir įrankių įsigijimo savikainos nusidėvėjimas (−)"
"account_account_template_12500","Cost of the Acquisition of Land as Investment Property","12500","asset_fixed","l10n_lt.account_account_tag_a_2_6_1","False","Žemės, kaip investicinio turto, įsigijimo savikaina"
"account_account_template_12509","Value Loss of the Land as Investment Property (-)","12509","asset_fixed","l10n_lt.account_account_tag_a_2_6_1","False","Žemės, kaip investicinio turto, vertės sumažėjimas (−)"
"account_account_template_12510","Cost of the Acquisition of Buildings as Investment Property","12510","asset_fixed","l10n_lt.account_account_tag_a_2_6_2","False","Pastatų, kaip investicinio turto, įsigijimo savikaina"
"account_account_template_12517","Depreciation of the Acquisition Cost of Buildings as Investment Property (-)","12517","asset_fixed","l10n_lt.account_account_tag_a_2_6_2","False","Pastatų, kaip investicinio turto, įsigijimo savikainos nusidėvėjimas (−)"
"account_account_template_12519","Value Loss of Buildings as Investment Property (-)","12519","asset_fixed","l10n_lt.account_account_tag_a_2_6_2","False","Pastatų, kaip investicinio turto, vertės sumažėjimas (−)"
"account_account_template_1260","Advance Payments for Long-Term Tangible Assets","1260","asset_fixed","l10n_lt.account_account_tag_a_2_7","False","Sumokėti avansai už ilgalaikį materialųjį turtą"
"account_account_template_166110","Acquisition Cost of Other Non-Equity Securities","166110","asset_non_current","l10n_lt.account_account_tag_a_3_7","False","Kitų ne nuosavybės vertybinių popierių įsigijimo savikaina"
"account_account_template_166119","Value Loss of Other Non-Equity Securities (-)","166119","asset_non_current","l10n_lt.account_account_tag_a_3_7","False","Kitų ne nuosavybės vertybinių popierių vertės sumažėjimas (−)"
"account_account_template_1665","Term Deposits","1665","asset_non_current","l10n_lt.account_account_tag_a_3_7","False","Terminuotieji indėliai"
"account_account_template_16700","Value of Receivable Debts from Customers","16700","asset_non_current","l10n_lt.account_account_tag_a_3_8","False","Gautinų pirkėjų skolų vertė"
"account_account_template_16710","Value of Loans Granted","16710","asset_non_current","l10n_lt.account_account_tag_a_3_8","False","Suteiktų paskolų vertė"
"account_account_template_1672","Receivable Amount of Leasing (Financial Rent) after One Year","1672","asset_non_current","l10n_lt.account_account_tag_a_3_8","False","Po vienų metų gautinos lizingo (finansinės nuomos) sumos"
"account_account_template_16740","Value of Receivables","16740","asset_non_current","l10n_lt.account_account_tag_a_3_8","False","Gautinų sumų vertė"
"account_account_template_1680","Advance Payments for Financial Assets","1680","asset_non_current","l10n_lt.account_account_tag_a_3_9","False","Sumokėti avansai už finansinį turtą"
"account_account_template_1681","Derivative Financial Assets","1681","asset_non_current","l10n_lt.account_account_tag_a_3_9","False","Iš išvestinių finansinių priemonių atsiradęs finansinis turtas"
"account_account_template_171","Deferred Profit Tax Asset","171","asset_non_current","l10n_lt.account_account_tag_a_4_1","False","Atidėtojo pelno mokesčio turtas"
"account_account_template_2010","Acquisition Cost of Raw Materials, Materials and Assembly Parts","2010","asset_current","l10n_lt.account_account_tag_b_1_1","False","Žaliavų, medžiagų ir komplektavimo detalių įsigijimo savikaina"
"account_account_template_2011","Raw Materials, Materials and Assembly Parts in Transit","2011","asset_current","l10n_lt.account_account_tag_b_1_1","True","Žaliavos, medžiagos ir komplektavimo detalės kelyje"
"account_account_template_2012","Raw Materials, Materials and Assembly Parts at Third Parties","2012","asset_current","l10n_lt.account_account_tag_b_1_1","False","Žaliavos, medžiagos ir komplektavimo detalės pas trečiuosius asmenis"
"account_account_template_2019","Value Loss of Raw Materials, Materials and Assembly Parts (-)","2019","asset_current","l10n_lt.account_account_tag_b_1_1","False","Žaliavų, medžiagų ir komplektavimo detalių vertės sumažėjimas (−)"
"account_account_template_20200","Cost of Incomplete Production","20200","asset_current","l10n_lt.account_account_tag_b_1_2","False","Nebaigtos produkcijos savikaina"
"account_account_template_20210","Cost of Executive Works","20210","asset_current","l10n_lt.account_account_tag_b_1_2","False","Vykdomų darbų savikaina"
"account_account_template_2030","Cost of Production","2030","asset_current","l10n_lt.account_account_tag_b_1_3","False","Produkcijos savikaina"
"account_account_template_2035","Production in Transit","2035","asset_current","l10n_lt.account_account_tag_b_1_3","True","Produkcija kelyje"
"account_account_template_2036","Production at Third Parties","2036","asset_current","l10n_lt.account_account_tag_b_1_3","False","Produkcija pas trečiuosius asmenis"
"account_account_template_2039","Value Loss of Production (-)","2039","asset_current","l10n_lt.account_account_tag_b_1_3","False","Produkcijos vertės sumažėjimas (−)"
"account_account_template_2040","Cost of Purchased Goods for Resale","2040","asset_current","l10n_lt.account_account_tag_b_1_4","False","Pirktų prekių, skirtų perparduoti, įsigijimo savikaina"
"account_account_template_2045","Purchased Goods for Resale in Transit","2045","asset_current","l10n_lt.account_account_tag_b_1_4","True","Pirktos prekės, skirtos perparduoti, kelyje"
"account_account_template_2046","Purchased Goods for Resale at Third Parties","2046","asset_current","l10n_lt.account_account_tag_b_1_4","False","Pirktos prekės, skirtos perparduoti, pas trečiuosius asmenis"
"account_account_template_2049","Value Loss of Purchased Goods for Resale (-)","2049","asset_current","l10n_lt.account_account_tag_b_1_4","False","Pirktų prekių, skirtų perparduoti, vertės sumažėjimas (−)"
"account_account_template_2060","Cost of Tangible Assets","2060","asset_current","l10n_lt.account_account_tag_b_1_6","False","Ilgalaikio materialiojo turto, skirto parduoti, savikaina"
"account_account_template_2080","Prepayments to Suppliers","2080","asset_prepayments","l10n_lt.account_account_tag_b_1_7","False","Sumokėti avansai tiekėjams"
"account_account_template_2084","Deposit","2084","asset_prepayments","l10n_lt.account_account_tag_b_1_7","False","Užstatas"
"account_account_template_2410","Value of Debts from Customers","2410","asset_receivable","l10n_lt.account_account_tag_b_2_1","True","Pirkėjų skolų vertė"
"account_account_template_2411","Value of Debts from POS Customers","2411","asset_receivable","l10n_lt.account_account_tag_b_2_1","True",""
"account_account_template_24400","Value of Loans Granted","24400","asset_receivable","l10n_lt.account_account_tag_b_2_4","True","Suteiktų paskolų vertė"
"account_account_template_2441","Receivable Value-Added Tax","2441","asset_current","l10n_lt.account_account_tag_b_2_4","False","Gautinas pridėtinės vertės mokestis"
"account_account_template_2442","Prepaid Profit Tax","2442","asset_receivable","l10n_lt.account_account_tag_b_2_4","True","Iš anksto sumokėtas pelno mokestis"
"account_account_template_2443","Tax Overpayments","2443","asset_receivable","l10n_lt.account_account_tag_b_2_4","True","Mokesčių permokos"
"account_account_template_2444","VSDF (Lithuanian State Social Insurance Fund) Debt to the Company","2444","asset_receivable","l10n_lt.account_account_tag_b_2_4","True","VSDF skola įmonei"
"account_account_template_24450","Value of Receivables from Accountable Persons","24450","asset_receivable","l10n_lt.account_account_tag_b_2_4","True","Iš atskaitingų asmenų gautinų sumų vertė"
"account_account_template_24460","Value of Other Receivable Debts","24460","asset_receivable","l10n_lt.account_account_tag_b_2_4","True","Kitų gautinos skolų vertė"
"account_account_template_24471","Profit paid in Advance to the Owners of the Company","24471","asset_receivable","l10n_lt.account_account_tag_b_2_4","True","Įmonės savininkams avansu išmokėtas pelnas"
"account_account_template_24472","Payed Funds for Personal Use to the Owners of the Company","24472","asset_receivable","l10n_lt.account_account_tag_b_2_4","True","Įmonės savininkų asmeniniams poreikiams išmokėtos lėšos"
"account_account_template_2449","Other Questionable Debts (-)","2449","asset_receivable","l10n_lt.account_account_tag_b_2_4","True","Kitos abejotinos skolos (−)"
"account_account_template_26200","Acquisition Cost of Equity Securities of Other Companies","26200","asset_current","l10n_lt.account_account_tag_b_3_2","False","Kitų įmonių nuosavybės vertybinių popierių įsigijimo savikaina"
"account_account_template_26209","Value Loss of Equity Securities of Other Companies (-)","26209","asset_current","l10n_lt.account_account_tag_b_3_2","False","Kitų įmonių nuosavybės vertybinių popierių vertės sumažėjimas (−)"
"account_account_template_262110","Acquisition Cost of Other Non-Equity Securities","262110","asset_current","l10n_lt.account_account_tag_b_3_2","False","Kitų ne nuosavybės vertybinių popierių įsigijimo savikaina"
"account_account_template_262119","Value Loss of Other Non-Equity Securities (-)","262119","asset_current","l10n_lt.account_account_tag_b_3_2","False","Kitų ne nuosavybės vertybinių popierių vertės sumažėjimas (−)"
"account_account_template_2623","Term Deposits","2623","asset_current","l10n_lt.account_account_tag_b_3_2","False","Terminuotieji indėliai"
"account_account_template_274","Cash Equivalents","274","asset_cash","l10n_lt.account_account_tag_b_4","False","Pinigų ekvivalentai"
"account_account_template_279","Frozen Funds (-)","279","asset_cash","l10n_lt.account_account_tag_b_4","False","Įšaldytos lėšos (−)"
"account_account_template_291","Future Expenses","291","asset_current","l10n_lt.account_account_tag_c_prepayments_accrued_income","False","Ateinančių laikotarpių sąnaudos"
"account_account_template_292","Accumulative Income","292","asset_current","l10n_lt.account_account_tag_c_prepayments_accrued_income","False","Sukauptos pajamos"
"account_account_template_3011","Ordinary Shares","3011","equity","l10n_lt.account_account_tag_d_1_1","False","Paprastosios akcijos"
"account_account_template_3012","Preference Shares","3012","equity","l10n_lt.account_account_tag_d_1_1","False","Privilegijuotosios akcijos"
"account_account_template_302","Subscribed Capital Unpaid (-)","302","equity","l10n_lt.account_account_tag_d_1_2","False","Pasirašytasis neapmokėtas kapitalas (−)"
"account_account_template_303","Own Shares (Yawn) (-)","303","equity","l10n_lt.account_account_tag_d_1_3","False","Savos akcijos (pajai) (−)"
"account_account_template_305","Company Owner’s Capital","305","equity","l10n_lt.account_account_tag_d_1_3","False","Įmonės savininko kapitalas"
"account_account_template_308","Owners’ Contributions","308","equity","l10n_lt.account_account_tag_d_1_3","False","Savininkų įnašai"
"account_account_template_331","Mandatory or Reserve Capital","331","equity","l10n_lt.account_account_tag_d_4_1","False","Privalomasis arba atsargos (rezervinis) kapitalas"
"account_account_template_3411","Acknowledged Net Profit (Loss) of The Accounting Year in The Profit (Loss) Report","3411","equity_unaffected","l10n_lt.account_account_tag_d_5_1","False","Pelno (nuostolių) ataskaitoje pripažintas ataskaitinių metų grynasis pelnas (nuostoliai)"
"account_account_template_3412","Unacknowledged Net Profit (Loss) of The Accounting Year in The Profit (Loss) Report","3412","equity","l10n_lt.account_account_tag_d_5_1","False","Pelno (nuostolių) ataskaitoje nepripažintas ataskaitinių metų pelnas (nuostoliai)"
"account_account_template_3421","Acknowledged Net Profit (Loss) of The Previous Accounting Year in The Profit (Loss) Report","3421","equity","l10n_lt.account_account_tag_d_5_2","False","Pelno (nuostolių) ataskaitoje pripažintas ankstesnių metų pelnas (nuostoliai)"
"account_account_template_3422","Unacknowledged Net Profit (Loss) of The Previous Accounting Year in The Profit (Loss) Report","3422","equity","l10n_lt.account_account_tag_d_5_2","False","Pelno (nuostolių) ataskaitoje nepripažintas ankstesnių metų pelnas (nuostoliai)"
"account_account_template_3424","Profit (Loss) of Material Corrections of The Previous Years","3424","equity","l10n_lt.account_account_tag_d_5_2","False","Ankstesnių metų esminių klaidų taisymo pelnas (nuostoliai)"
"account_account_template_390","Common Summary of Accounts","390","equity","l10n_lt.account_account_tag_d_6","False","Bendra sąskaitų suvestinė"
"account_account_template_411","Provisions for Pensions and Similar Liabilities","411","liability_non_current","l10n_lt.account_account_tag_f_1","False","Pensijų ir panašių įsipareigojimų atidėjiniai"
"account_account_template_412","Tax Provisions","412","liability_non_current","l10n_lt.account_account_tag_f_2","False","Mokesčių atidėjiniai"
"account_account_template_413","Other Provisions","413","liability_non_current","l10n_lt.account_account_tag_f_3","False","Kiti atidėjiniai"
"account_account_template_4211","Leasing (Financial Rent) or Other Obligations","4211","liability_non_current","l10n_lt.account_account_tag_g_1_1","False","Lizingo (finansinės nuomos) ar panašūs įsipareigojimai"
"account_account_template_4214","Other Debt Obligations","4214","liability_non_current","l10n_lt.account_account_tag_g_1_1","False","Kiti skoliniai įsipareigojimai"
"account_account_template_4220","Long-Term Liabilities under Loan Agreements","4220","liability_non_current","l10n_lt.account_account_tag_g_1_2","False","Ilgalaikiai įsipareigojimai pagal paskolų sutartis"
"account_account_template_4221","Other Obligations","4221","liability_non_current","l10n_lt.account_account_tag_g_1_2","False","Kiti įsipareigojimai"
"account_account_template_4230","Prepayments from Customers","4230","liability_non_current","l10n_lt.account_account_tag_g_1_3","False","Iš pirkėjų gauti avansai"
"account_account_template_4231","Prepayments from Service Receivers","4231","liability_non_current","l10n_lt.account_account_tag_g_1_3","False","Iš paslaugų gavėjų gauti avansai"
"account_account_template_424","Debts to Suppliers","424","liability_non_current","l10n_lt.account_account_tag_g_1_4","False","Skolos tiekėjams"
"account_account_template_428","Other Payables and Long-Term Liabilities","428","liability_non_current","l10n_lt.account_account_tag_g_1_8","False","Kitos mokėtinos sumos ir ilgalaikiai įsipareigojimai"
"account_account_template_4401","Part of Leasing (Financial Rent) or Other Obligations in The Current Year","4401","liability_current","l10n_lt.account_account_tag_g_2_1","False","Lizingo (finansinės nuomos) ar panašių įsipareigojimų einamųjų metų dalis"
"account_account_template_4403","Part of Other Long-Term Debts in The Current Year","4403","liability_current","l10n_lt.account_account_tag_g_2_1","False","Kitų ilgalaikių skolų einamųjų metų dalis"
"account_account_template_4410","Liabilities under Short-Term Loan Agreements","4410","liability_current","l10n_lt.account_account_tag_g_2_2","False","Įsipareigojimai pagal trumpalaikių paskolų sutartis"
"account_account_template_4411","Part of Debts to Credit Institutions in The Current Year","4411","liability_current","l10n_lt.account_account_tag_g_2_2","False","Skolų kredito įstaigoms einamųjų metų dalis"
"account_account_template_4413","Other Obligations (Executables and etc.)","4413","liability_current","l10n_lt.account_account_tag_g_2_2","False","Kiti įsipareigojimai (vykdomieji ir pan.)"
"account_account_template_4420","Prepayments from Customers","4420","liability_current","l10n_lt.account_account_tag_g_2_3","False","Iš pirkėjų gauti avansai"
"account_account_template_4421","Prepayments from Service Receivers","4421","liability_current","l10n_lt.account_account_tag_g_2_3","False","Iš paslaugų gavėjų gauti avansai"
"account_account_template_4430","Debts to Suppliers for Goods and Services","4430","liability_payable","l10n_lt.account_account_tag_g_2_4","True","Skolos tiekėjams už prekes ir paslaugas"
"account_account_template_4431","Other Debts to Suppliers","4431","liability_payable","l10n_lt.account_account_tag_g_2_4","True","Kitos skolos tiekėjams"
"account_account_template_4470","Obligations of Profit Tax","4470","liability_current","l10n_lt.account_account_tag_g_2_8","False","Pelno mokesčio įsipareigojimai"
"account_account_template_4471","Other Obligations","4471","liability_current","l10n_lt.account_account_tag_g_2_8","False","Kiti įsipareigojimai"
"account_account_template_4480","Payable Salary","4480","liability_current","l10n_lt.account_account_tag_g_2_9","False","Mokėtinas darbo užmokestis"
"account_account_template_4481","Payable Personal Income Tax","4481","liability_current","l10n_lt.account_account_tag_g_2_9","False","Mokėtinas gyventojų pajamų mokestis"
"account_account_template_4482","Contributions of Payable Social Security","4482","liability_current","l10n_lt.account_account_tag_g_2_9","False","Mokėtinos socialinio draudimo įmokos"
"account_account_template_4483","Contributions of Payable Guarantee Fund","4483","liability_current","l10n_lt.account_account_tag_g_2_9","False","Mokėtinos garantinio fondo įmokos"
"account_account_template_4485","Leave Liabilities","4485","liability_current","l10n_lt.account_account_tag_g_2_9","False","Atostoginių kaupiniai"
"account_account_template_4486","Contributions of Payable Compulsory Health Insurance","4486","liability_current","l10n_lt.account_account_tag_g_2_9","False","Mokėtinos privalomojo sveikatos draudimo įmokos"
"account_account_template_4490","Payable Dividends","4490","liability_current","l10n_lt.account_account_tag_g_2_10","False","Mokėtini dividendai"
"account_account_template_4491","Payable Bonuses","4491","liability_current","l10n_lt.account_account_tag_g_2_10","False","Mokėtinos tantjemos"
"account_account_template_4492","Payable Value-Added Tax","4492","liability_current","l10n_lt.account_account_tag_g_2_10","False","Mokėtinas pridėtinės vertės mokestis"
"account_account_template_4493","Other Payable Taxes to Budget","4493","liability_current","l10n_lt.account_account_tag_g_2_10","False","Kiti į biudžetą mokėtini mokesčiai"
"account_account_template_4494","Other Payables","4494","liability_current","l10n_lt.account_account_tag_g_2_10","False","Kitos mokėtinos sumos"
"account_account_template_4495","Payables to the Owners of the Company","4495","liability_current","l10n_lt.account_account_tag_g_2_10","False","Įmonės savininkams mokėtinos sumos"
"account_account_template_491","Accumulated Expenses","491","liability_current","l10n_lt.account_account_tag_h_accruals_deferred_income","False","Sukauptos sąnaudos"
"account_account_template_492","Income of Future Periods","492","liability_current","l10n_lt.account_account_tag_h_accruals_deferred_income","False","Ateinančių laikotarpių pajamos"
"account_account_template_5000","Income from Goods Sold","5000","income","l10n_lt.account_account_tag_1_net_turnover","False","Parduotų prekių pajamos"
"account_account_template_5001","Income from Services Sold","5001","income","l10n_lt.account_account_tag_1_net_turnover","False","Suteiktų paslaugų pajamos"
"account_account_template_5003","Income from Production Sold","5003","income","l10n_lt.account_account_tag_1_net_turnover","False","Pagamintos produkcijos pajamos"
"account_account_template_509","Discounts, Returns (-)","509","income","l10n_lt.account_account_tag_1_net_turnover","False","Nuolaidos, grąžinimas (−)"
"account_account_template_5400","Profit from Asset Disposition","5400","income_other","l10n_lt.account_account_tag_6_other_operating_results","False","Ilgalaikio turto perleidimo pelnas"
"account_account_template_5401","Other Income","5401","income_other","l10n_lt.account_account_tag_6_other_operating_results","False","Kitos pajamos"
"account_account_template_5600","Income from Other Long-Term Investments and Loan Interests","5600","income_other","l10n_lt.account_account_tag_8_income_other_longterm_investments_loans","False","Kitų ilgalaikių investicijų ir paskolų palūkanų pajamos"
"account_account_template_5802","Income from Other Loan Interests","5802","income_other","l10n_lt.account_account_tag_9_other_interest_similar_income","False","Kitų suteiktų paskolų palūkanų pajamos"
"account_account_template_5803","Positive Currency Exchange Difference","5803","income","l10n_lt.account_account_tag_9_other_interest_similar_income,account.account_tag_financing","False","Teigiama valiutų kursų pokyčio įtaka"
"account_account_template_5804","Income from Fines","5804","income_other","l10n_lt.account_account_tag_9_other_interest_similar_income","False","Baudų ir delspinigių pajamos"
"account_account_template_5809","Profit from Disposition of Investments","5809","income_other","l10n_lt.account_account_tag_9_other_interest_similar_income","False","Investicijų perleidimo pelnas"
"account_account_template_6000","Cost of Goods Sold","6000","expense_direct_cost","l10n_lt.account_account_tag_2_cost_of_sales","False","Parduotų prekių savikaina"
"account_account_template_6001","Cost of Services Sold","6001","expense_direct_cost","l10n_lt.account_account_tag_2_cost_of_sales","False","Suteiktų paslaugų savikaina"
"account_account_template_60030","Cost of Production Sold","60030","expense_direct_cost","l10n_lt.account_account_tag_2_cost_of_sales","False","Parduotos pagamintos produkcijos savikaina"
"account_account_template_60031","Short-Term Production Inventory","60031","expense_direct_cost","l10n_lt.account_account_tag_2_cost_of_sales","False","Mažavertis gamybos inventorius"
"account_account_template_60032","Direct Transport","60032","expense_direct_cost","l10n_lt.account_account_tag_2_cost_of_sales","False","Tiesioginis transportas"
"account_account_template_60033","Direct Commissions","60033","expense_direct_cost","l10n_lt.account_account_tag_2_cost_of_sales","False","Tiesioginiai komisiniai"
"account_account_template_60034","Direct Payroll of Employees Working in Production","60034","expense_direct_cost","l10n_lt.account_account_tag_2_cost_of_sales","False","Tiesioginis gamybos DU"
"account_account_template_60040","Working Tools for Production","60040","expense_direct_cost","l10n_lt.account_account_tag_2_cost_of_sales","False","Gamybos įrankiai ir darbo priemonės"
"account_account_template_60041","Maintenance of Working Tools for Production","60041","expense_direct_cost","l10n_lt.account_account_tag_2_cost_of_sales","False","Gamybos įrangos remontas ir techn. aptarnavimas"
"account_account_template_60042","Maintenance of Production Transport","60042","expense_direct_cost","l10n_lt.account_account_tag_2_cost_of_sales","False","Gamybos transporto remontas ir techn. aptarnavimas"
"account_account_template_60043","Energy Resources","60043","expense_direct_cost","l10n_lt.account_account_tag_2_cost_of_sales","False","Energetiniai resursai"
"account_account_template_60044","Fuel","60044","expense_direct_cost","l10n_lt.account_account_tag_2_cost_of_sales","False","Kuras"
"account_account_template_6005","Increase (Decrease) in Inventories","6005","expense_direct_cost","l10n_lt.account_account_tag_2_cost_of_sales","False","Atsargų padidėjimas (sumažėjimas)"
"account_account_template_609","Discounts, Returns (-)","609","expense_direct_cost","l10n_lt.account_account_tag_2_cost_of_sales","False","Nuolaidos, grąžinimas (−)"
"account_account_template_6202","Cost of Services and Goods Advertising","6202","expense","l10n_lt.account_account_tag_4_selling_expenses","False","Paslaugų ir prekių reklamos sąnaudos"
"account_account_template_6203","Payroll and Other Costs of Employees Working in Sales","6203","expense","l10n_lt.account_account_tag_4_selling_expenses","False","Pardavimo darbuotojų DU ir su juo susijusios sąnaudos"
"account_account_template_6208","Other Costs of Sales","6208","expense","l10n_lt.account_account_tag_4_selling_expenses","False","Kitos pardavimo sąnaudos"
"account_account_template_6209","Discounts Received (-)","6209","expense","l10n_lt.account_account_tag_4_selling_expenses","False","Gautos nuolaidos (−)"
"account_account_template_6300","Rental Costs","6300","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Nuomos sąnaudos"
"account_account_template_6301","Costs of Maintenance and Exploitation","6301","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Remonto ir eksploatacijos sąnaudos"
"account_account_template_6302","Fuel Costs","6302","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Kuro sąnaudos"
"account_account_template_6303","Insurance Costs","6303","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Draudimo sąnaudos"
"account_account_template_6304","Payroll and Other Costs of Employees Working in Administration","6304","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Administracijos darbuotojų DU ir su juo susijusios sąnaudos"
"account_account_template_6306","Depreciation Costs of Long-Term Tangible Assets","6306","expense_depreciation","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Ilgalaikio materialiojo turto vertės nusidėvėjimo sąnaudos"
"account_account_template_6307","Amortization Costs of Intangible Assets Value","6307","expense_depreciation","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Nematerialiojo turto vertės amortizacijos sąnaudos"
"account_account_template_63080","Costs of Real Estate Tax","63080","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Nekilnojamojo turto mokesčio sąnaudos"
"account_account_template_63081","Costs of not Deductible Value-Added Tax","63081","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Neatskaitomo pridėtinės vertės mokesčio sąnaudos"
"account_account_template_63082","Pollution Tax Costs","63082","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Aplinkos teršimo mokesčio sąnaudos"
"account_account_template_63083","Costs of Other Taxes","63083","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Kitų mokesčių sąnaudos"
"account_account_template_63090","Costs of Value Loss of Debts from Customers","63090","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Pirkėjų skolų vertės sumažėjimo sąnaudos"
"account_account_template_63091","Costs of Inventory Value Loss","63091","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Atsargų vertės sumažėjimo sąnaudos"
"account_account_template_63092","Costs of Intangible Assets Value Loss","63092","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Nematerialiojo turto vertės sumažėjimo sąnaudos"
"account_account_template_63093","Costs of Value Loss of Long-Term Tangible Assets","63093","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Ilgalaikio materialiojo turto vertės sumažėjimo sąnaudos"
"account_account_template_63094","Costs of Value Loss of Other Assets","63094","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Kito turto vertės sumažėjimo sąnaudos"
"account_account_template_6310","Provisions Costs","6310","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Atidėjinių sąnaudos"
"account_account_template_6311","Costs of Fines","6311","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Baudų ir delspinigių sąnaudos"
"account_account_template_63120","Other Costs and Bank Taxes","63120","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Kitos sąnaudos ir banko mokesčiai"
"account_account_template_63121","Representative Costs","63121","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Reprezentacinės sąnaudos"
"account_account_template_63122","Daily Allowance","63122","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Dienpinigiai"
"account_account_template_63123","Support","63123","expense","l10n_lt.account_account_tag_5_general_administrative_expenses","False","Parama"
"account_account_template_6400","Asset Disposition Losses","6400","expense","l10n_lt.account_account_tag_6_other_operating_results","False","Ilgalaikio turto perleidimo nuostoliai"
"account_account_template_6401","Other Costs","6401","expense","l10n_lt.account_account_tag_6_other_operating_results","False","Kitos sąnaudos"
"account_account_template_6701","Costs of Value Loss of Long-Term Financial Assets","6701","expense","l10n_lt.account_account_tag_10_impaired_fin_assets_short_investments","False","Ilgalaikio finansinio turto vertės sumažėjimo sąnaudos"
"account_account_template_6702","Costs of Value Loss of Short-Term Investments","6702","expense","l10n_lt.account_account_tag_10_impaired_fin_assets_short_investments","False","Trumpalaikių investicijų vertės sumažėjimo sąnaudos"
"account_account_template_6802","Costs of Other Loan Interests","6802","expense","l10n_lt.account_account_tag_11_interest_other_similar_expenses","False","Kitų suteiktų paskolų palūkanų sąnaudos"
"account_account_template_6803","Negative Currency Exchange Difference","6803","expense","l10n_lt.account_account_tag_11_interest_other_similar_expenses","False","Neigiama valiutų kursų pokyčio įtaka"
"account_account_template_6804","Costs of Fines","6804","expense","l10n_lt.account_account_tag_11_interest_other_similar_expenses","False","Baudų ir delspinigių sąnaudos"
"account_account_template_6805","Representation and other disallowed deductions","6805","expense","l10n_lt.account_account_tag_11_interest_other_similar_expenses","False","Reprezentacija ir kiti neleidžiami atskaitymai"
"account_account_template_6806","Costs of Interests for Financial Rent","6806","expense","l10n_lt.account_account_tag_11_interest_other_similar_expenses","False","Palūkanų sąnaudos už finansinės nuomos būdu įsigyjamą turtą"
"account_account_template_6809","Investment Disposition Losses","6809","expense","l10n_lt.account_account_tag_11_interest_other_similar_expenses","False","Investicijų perleidimo nuostoliai"
"account_account_template_6900","Profit and Other Similar Taxes of The Current Year","6900","income","l10n_lt.account_account_tag_12_tax_on_profit","False","Ataskaitinių metų pelno ir panašūs mokesčiai"
"account_account_template_6901","Costs (Income) of Deferred Value-Added Tax","6901","expense","l10n_lt.account_account_tag_12_tax_on_profit","False","Atidėtojo pelno mokesčio sąnaudos (pajamos)"
"account_chart_template_lithuania_liquidity_transfer","Cash in Transit","273","asset_current","l10n_lt.account_account_tag_b_4","True","Pinigai kelyje"
"account_account_template_999","Interim Account","999","asset_cash","l10n_lt.account_account_tag_b_4","False","Tarpinė sąskaita užskaitoms"

```

## File: data\template\account.fiscal.position-lt.csv

```csv
"id","name","auto_apply","vat_required","country_id","sequence","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@lt"
"account_fiscal_position_template_lt","LT","1","1","base.lt","10","","","","LT"
"account_fiscal_position_template_eu","EU","1","1","","20","base.europe","account_tax_template_sales_21","account_tax_template_sales_0_vat13","ES"
"","","","","","","","account_tax_template_purchase_21","account_tax_template_purchase_assumed_21",""
"account_fiscal_position_template_b2c_eu","B2C EU","1","0","","25","base.europe","","","B2C ES"
"account_fiscal_position_template_out_eu","Not EU","1","","","30","","account_tax_template_sales_21","account_tax_template_sales_0_vat12","Ne ES"
"","","","","","","","account_tax_template_purchase_21","account_tax_template_purchase_0",""

```

## File: data\template\account.tax-lt.csv

```csv
"id","tax_group_id","name","type_tax_use","amount","amount_type","sequence","description","invoice_label","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@lt"
"account_tax_template_sales_0_vat5","tax_group_vat_0","0% EXEMPT","sale","0.0","percent","10","Sale 0% (VAT5) Tax Exempt in LT","","base","invoice","","","Pardavimo 0% (PVM5) neapmokestinamas LT"
"","","","","","","","","","tax","invoice","account_account_template_4492","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_4492","",""
"account_tax_template_sales_0_vat12","tax_group_vat_0","0% EX","sale","0.0","percent","10","Sale 0% (VAT12) export","0% VAT","base","invoice","","","Pardavimo 0% (PVM12) eksportas"
"","","","","","","","","","tax","invoice","account_account_template_4492","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_4492","",""
"account_tax_template_sales_0_vat13","tax_group_vat_0","0%","sale","0.0","percent","10","Sale 0% (VAT13)","0% VAT","base","invoice","","","Pardavimo 0% (PVM13)"
"","","","","","","","","","tax","invoice","account_account_template_4492","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_4492","",""
"account_tax_template_sales_0_vat15","tax_group_vat_0","0% EX S","sale","0.0","percent","10","Sale 0% (VAT15) Service not in EU","","base","invoice","","","Pardavimo 0% (PVM15) paslaugos ne ES"
"","","","","","","","","","tax","invoice","account_account_template_4492","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_4492","",""
"account_tax_template_sales_5","tax_group_vat_5","5%","sale","5.0","percent","10","Sale 5% (VAT3)","5% VAT","base","invoice","","","Pardavimo 5% (PVM3)"
"","","","","","","","","","tax","invoice","account_account_template_4492","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_4492","",""
"account_tax_template_sales_21","tax_group_vat_21","21%","sale","21.0","percent","1","Sale 21% (VAT1)","21% VAT","base","invoice","","","Pardavimo 21% (PVM1)"
"","","","","","","","","","tax","invoice","account_account_template_4492","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_4492","",""
"account_tax_template_sales_reversed_21","tax_group_vat_21","21% R","sale","21.0","percent","10","Reversed Sale 21% (VAT25)","21% VAT","base","invoice","","","Atvirkštinis pardavimo 21% (PVM25)"
"","","","","","","","","","tax","invoice","account_account_template_4492","",""
"","","","","","","","","","tax","invoice","account_account_template_4492","-100",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_4492","",""
"","","","","","","","","","tax","refund","account_account_template_4492","-100",""
"account_tax_template_purchase_0_vat5","tax_group_vat_0","0% EXEMPT","purchase","0.0","percent","10","Purchase 0% (VAT5) Tax Exempt LT","","base","invoice","","","Pirkimo 0% (PVM5) neapmokestinamas LT"
"","","","","","","","","","tax","invoice","account_account_template_2441","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_2441","",""
"account_tax_template_purchase_0_vat14","tax_group_vat_0","0% EX","purchase","0.0","percent","10","Purchase 0% (VAT14) export, transportation","0% VAT","base","invoice","","","Pirkimo 0% (PVM14) eksportas, transportavimas"
"","","","","","","","","","tax","invoice","account_account_template_2441","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_2441","",""
"account_tax_template_purchase_0","tax_group_vat_0","0% VAT15","purchase","0.0","percent","10","Purchase 0% (VAT15)","0% VAT","base","invoice","","","Pirkimo 0% (PVM15)"
"","","","","","","","","","tax","invoice","account_account_template_2441","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_2441","",""
"account_tax_template_purchase_0_vat42","tax_group_vat_0","0% VAT42","purchase","0.0","percent","10","Purchase 0% (VAT42)","","base","invoice","","","Pirkimo 0% (PVM42)"
"","","","","","","","","","tax","invoice","account_account_template_2441","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_2441","",""
"account_tax_template_purchase_0_vat100","tax_group_vat_0","0% VAT100","purchase","0.0","percent","10","Purchase 0% (VAT100) other cases","","base","invoice","","","Pirkimo 0% (PVM100) kiti atvejai"
"","","","","","","","","","tax","invoice","account_account_template_2441","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_2441","",""
"account_tax_template_purchase_5","tax_group_vat_5","5%","purchase","5.0","percent","10","Purchase 5% (VAT3)","5% VAT","base","invoice","","","Pirkimo 5% (PVM3)"
"","","","","","","","","","tax","invoice","account_account_template_2441","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_2441","",""
"account_tax_template_purchase_not_deductible_9","tax_group_vat_9","9% ND","purchase","9.0","percent","10","Purchase 9% (VAT2) not deductible","9% VAT","base","invoice","","","Pirkimo 9% (PVM2) neatskaitomas"
"","","","","","","","","","tax","invoice","account_account_template_63081","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_63081","",""
"account_tax_template_purchase_9","tax_group_vat_9","9%","purchase","9.0","percent","10","Purchase 9% (VAT2)","9% VAT","base","invoice","","","Pirkimo 9% (PVM2)"
"","","","","","","","","","tax","invoice","account_account_template_2441","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_2441","",""
"account_tax_template_purchase_not_deductible_21","tax_group_vat_21","21% ND","purchase","21.0","percent","10","Purchase 21% (VAT1) not deductible","21% VAT","base","invoice","","","Pirkimo 21% (PVM1) neatskaitomas"
"","","","","","","","","","tax","invoice","account_account_template_63081","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_63081","",""
"account_tax_template_purchase_21","tax_group_vat_21","21%","purchase","21.0","percent","1","Purchase 21% (VAT1)","21% VAT","base","invoice","","","Pirkimo 21% (PVM1)"
"","","","","","","","","","tax","invoice","account_account_template_2441","",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_2441","",""
"account_tax_template_purchase_reversed_21","tax_group_vat_21","21% R","purchase","21.0","percent","90","Reversed Purchase 21% (VAT25)","21% VAT","base","invoice","","","Atvirkštinis pirkimo 21% (PVM25)"
"","","","","","","","","","tax","invoice","account_account_template_4492","",""
"","","","","","","","","","tax","invoice","account_account_template_4492","-100",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_4492","",""
"","","","","","","","","","tax","refund","account_account_template_4492","-100",""
"account_tax_template_purchase_assumed_21_vat16","tax_group_vat_21","21% EU A","purchase","21.0","percent","100","Assumed Purchase 21% (VAT16) from EU","Assumed 21% VAT","base","invoice","","","Menamas pirkimo 21% (PVM16) iš ES"
"","","","","","","","","","tax","invoice","account_account_template_2441","",""
"","","","","","","","","","tax","invoice","account_account_template_4492","-100",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_2441","",""
"","","","","","","","","","tax","refund","account_account_template_4492","-100",""
"account_tax_template_purchase_assumed_21_vat20","tax_group_vat_21","21% A","purchase","21.0","percent","110","Assumed Purchase 21% (VAT20)","Assumed 21% VAT","base","invoice","","","Menamas pirkimo 21% PVM (PVM20)"
"","","","","","","","","","tax","invoice","account_account_template_2441","",""
"","","","","","","","","","tax","invoice","account_account_template_4492","-100",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_2441","",""
"","","","","","","","","","tax","refund","account_account_template_4492","-100",""
"account_tax_template_purchase_assumed_21","tax_group_vat_21","21% G S","purchase","21.0","percent","120","Assumed Purchase 21% (VAT21) Goods/Services","Assumed 21% VAT","base","invoice","","","Menamas pirkimo 21% PVM (PVM21) prekių/paslaugų"
"","","","","","","","","","tax","invoice","account_account_template_2441","",""
"","","","","","","","","","tax","invoice","account_account_template_4492","-100",""
"","","","","","","","","","base","refund","","",""
"","","","","","","","","","tax","refund","account_account_template_2441","",""
"","","","","","","","","","tax","refund","account_account_template_4492","-100",""

```

## File: data\template\account.tax.group-lt.csv

```csv
"id","name","country_id","name@lt"
"tax_group_vat_0","VAT 0%","base.lt","PVM 0%"
"tax_group_vat_5","VAT 5%","base.lt","PVM 5%"
"tax_group_vat_9","VAT 9%","base.lt","PVM 9%"
"tax_group_vat_21","VAT 21%","base.lt","PVM 21%"

```

## File: i18n_extra\l10n_lt.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
#   * l10n_lt
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 12.0+e\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2019-09-05 09:02+0000\n"
"PO-Revision-Date: 2019-09-05 09:02+0000\n"
"Last-Translator: <>\n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_1_net_turnover
msgid "1. Net turnover"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_9_other_interest_similar_income
msgid "9. Other interest and similar income"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_10_impaired_fin_assets_short_investments
msgid "10. The impairment of the financial assets and short-term investments"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_11_interest_other_similar_expenses
msgid "11. Interest and other similar expenses"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_12_tax_on_profit
msgid "12. Tax on profit"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_2_cost_of_sales
msgid "2. Cost of sales"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_3_adjustments_of_biological_assets
msgid "3. Fair value adjustments of the biological assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_4_selling_expenses
msgid "4. Selling expenses"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_5_general_administrative_expenses
msgid "5. General and administrative expenses"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_6_other_operating_results
msgid "6. Other operating results"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_7_income_investments_parent
msgid ""
"7. Income from investments in the shares of parent, subsidiaries and "
"associated entities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_8_income_other_longterm_investments_loans
msgid "8. Income from other long-term investments and loans"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_1_1
msgid "A.1.1. Assets arising from development"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_1_2
msgid "A.1.2. Goodwill"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_1_3
msgid "A.1.3. Software"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_1_4
msgid "A.1.4. Concessions, patents, licenses, trade marks and similar rights"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_1_5
msgid "A.1.5. Other intangible assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_1_6
msgid "A.1.6. Advance payments"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_1
msgid "A.2.1. Land"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_2
msgid "A.2.2. Buildings and structures"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_3
msgid "A.2.3. Machinery and plant"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_4
msgid "A.2.4. Vehicles"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_5
msgid "A.2.5. Other equipment, fittings and tools"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_6_1
msgid "A.2.6.1. Land"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_6_2
msgid "A.2.6.2. Buildings"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_2_7
msgid ""
"A.2.7. Advance payments and tangible assets under construction (production)"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_1
msgid "A.3.1. Shares in entities of the entities group"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_2
msgid "A.3.2. Loans to entities of the entities group"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_3
msgid "A.3.3. Amounts receivable from entities of the entities group"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_4
msgid "A.3.4. Shares in associated entities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_5
msgid "A.3.5. Loans to associated entities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_6
msgid "A.3.6. Amounts receivable from the associated entities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_7
msgid "A.3.7. Long-term investments"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_8
msgid "A.3.8. Amounts receivable after one year"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_3_9
msgid "A.3.9. Other financial assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_4_1
msgid "A.4.1. Assets of the deferred tax on profit"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_4_2
msgid "A.4.2. Biological assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_a_4_3
msgid "A.4.3. Other assets"
msgstr ""

#. module: l10n_lt
#: model:ir.model,name:l10n_lt.model_account_chart_template
msgid "Account Chart Template"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_1_1
msgid "B.1.1. Raw materials, materials and consumables"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_1_2
msgid "B.1.2. Production and work in progress"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_1_3
msgid "B.1.3. Finished goods"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_1_4
msgid "B.1.4. Goods for resale"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_1_5
msgid "B.1.5. Biological assets"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_1_6
msgid "B.1.6. Fixed tangible assets held for sale"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_1_7
msgid "B.1.7. Advance payments"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_2_1
msgid "B.2.1. Trade debtors"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_2_2
msgid "B.2.2. Amounts owed by entities of the entities group"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_2_3
msgid "B.2.3. Amounts owed by associates entities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_2_4
msgid "B.2.4. Other debtors"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_3_1
msgid "B.3.1. Shares in entities of the entities group"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_3_2
msgid "B.3.2. Other investments"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_b_4
msgid "B.4. CASH AND CASH EQUIVALENTS"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_c_prepayments_accrued_income
msgid "C. PREPAYMENTS AND ACCRUED INCOME"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_1_1
msgid "D.1.1. Authorized (subscribed) or primary capital"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_1_2
msgid "D.1.2. Subscribed capital unpaid (–)"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_1_3
msgid "D.1.3. Own shares (–)"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_2_share_premium
msgid "D.2. SHARE PREMIUM ACCOUNT"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_3_revaluation_reserve
msgid "D.3. REVALUATION RESERVE"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_4_1
msgid "D.4.1. Compulsory reserve or emergency (reserve) capital"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_4_2
msgid "D.4.2. Reserve for acquiring own shares"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_4_3
msgid "D.4.3. Other reserves"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_5_1
msgid "D.5.1. Profit (loss) for the reporting year "
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_5_2
msgid "D.5.2. Profit (loss) brought forward"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_d_6
msgid "D.6. Common Summary of Accounts"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_e_grants_subsidies
msgid "E. GRANTS, SUBSIDIES"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_f_1
msgid "F.1. Provisions for pensions and similar obligations"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_f_2
msgid "F.2. Provisions for taxation"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_f_3
msgid "F.3. Other provisions"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_1
msgid "G.1.1. Debenture loans"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_2
msgid "G.1.2. Amounts owed to credit institutions"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_3
msgid "G.1.3. Payments received on account"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_4
msgid "G.1.4. Trade creditors"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_5
msgid "G.1.5. Amounts payable under the bills and checks "
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_6
msgid "G.1.6. Amounts payable to the entities of the entities group"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_7
msgid "G.1.7. Amounts payable to the associated entities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_1_8
msgid "G.1.8. Other amounts payable and long-term liabilities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_1
msgid "G.2.1. Debenture loans"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_10
msgid "G.2.10. Other amounts payable and short-term liabilities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_2
msgid "G.2.2. Amounts owed to credit institutions"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_3
msgid "G.2.3. Payments received on account"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_4
msgid "G.2.4. Trade creditors"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_5
msgid "G.2.5. Amounts payable under the bills and checks "
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_6
msgid "G.2.6. Amounts payable to the entities of the entities group"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_7
msgid "G.2.7. Amounts payable to the associated entities"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_8
msgid "G.2.8. Liabilities of tax on profit"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_g_2_9
msgid "G.2.9. Liabilities related to employment relations"
msgstr ""

#. module: l10n_lt
#: model:account.account.tag,name:l10n_lt.account_account_tag_h_accruals_deferred_income
msgid "H. ACCRUALS AND DEFERRED INCOME"
msgstr ""

#. module: l10n_lt
#: model:ir.model,name:l10n_lt.model_account_journal
msgid "Journal"
msgstr ""

```

## File: models\account_journal.py

```python
from odoo import api, models, Command


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    @api.model
    def _prepare_liquidity_account_vals(self, company, code, vals):
        ''' Set tags on new bank and cash accounts.'''
        # OVERRIDE
        account_vals = super()._prepare_liquidity_account_vals(company, code, vals)

        if company.account_fiscal_country_id.code == 'LT':
            account_vals.setdefault('tag_ids', [])
            account_vals['tag_ids'] += [
                Command.link(self.env.ref('l10n_lt.account_account_tag_b_4').id),
            ]

        return account_vals

```

## File: models\template_lt.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('lt')
    def _get_lt_template_data(self):
        return {
            'property_account_receivable_id': 'account_account_template_2410',
            'property_account_payable_id': 'account_account_template_4430',
            'property_account_expense_categ_id': 'account_account_template_6000',
            'property_account_income_categ_id': 'account_account_template_5000',
            'property_stock_account_input_categ_id': 'account_account_template_2045',
            'property_stock_account_output_categ_id': 'account_account_template_2045',
            'property_stock_valuation_account_id': 'account_account_template_2040',
            'code_digits': '6',
        }

    @template('lt', 'res.company')
    def _get_lt_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.lt',
                'bank_account_code_prefix': '271',
                'cash_account_code_prefix': '272',
                'transfer_account_code_prefix': '273',
                'account_default_pos_receivable_account_id': 'account_account_template_2411',
                'income_currency_exchange_account_id': 'account_account_template_5803',
                'expense_currency_exchange_account_id': 'account_account_template_6803',
                'account_journal_early_pay_discount_loss_account_id': 'account_account_template_509',
                'account_journal_early_pay_discount_gain_account_id': 'account_account_template_6209',
                'account_sale_tax_id': 'account_tax_template_sales_21',
                'account_purchase_tax_id': 'account_tax_template_purchase_21',
            },
        }
    def _setup_utility_bank_accounts(self, template_code, company, template_data):
        super()._setup_utility_bank_accounts(template_code, company, template_data)
        if template_code == "lt":
            bank_tags = self.env.ref('l10n_lt.account_account_tag_b_4')
            company.account_journal_suspense_account_id.tag_ids |= bank_tags
            company.transfer_account_id.tag_ids |= bank_tags

            other_operating_results_tags = self.env.ref('l10n_lt.account_account_tag_6_other_operating_results')
            company.default_cash_difference_income_account_id.tag_ids |= other_operating_results_tags
            company.default_cash_difference_expense_account_id.tag_ids |= other_operating_results_tags

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_journal
from . import template_lt

```

