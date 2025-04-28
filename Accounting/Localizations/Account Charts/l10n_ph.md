# Odoo Module: l10n_ph

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import wizard

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Philippines - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['ph'],
    'summary': "This is the module to manage the accounting chart for The Philippines.",
    'category': 'Accounting/Localizations/Account Charts',
    'version': '1.1',
    'author': 'Odoo PS',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations/philippines.html',
    'depends': [
        'account',
        'base_vat',
    ],
    'data': [
        'data/account_account_tag_data.xml',
        'data/account_tax_report_data.xml',
        'wizard/generate_2307_wizard_views.xml',
        'views/account_move_views.xml',
        'views/account_payment_views.xml',
        'views/account_tax_views.xml',
        'views/res_partner_views.xml',
        'security/ir.model.access.csv',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_account_tag_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <!-- required for the SLSP but cannot yet be mapped to the tax report as we don't support amortized tax yet -->
    <record id="tax_tag_capital_base_positive" model="account.account.tag">
        <field name="name">+CAPA</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.ph"/>
    </record>
    <record id="tax_tag_capital_base_negative" model="account.account.tag">
        <field name="name">-CAPA</field>
        <field name="applicability">taxes</field>
        <field name="tax_negate">True</field>
        <field name="country_id" ref="base.ph"/>
    </record>
    <record id="tax_tag_capital_tax_positive" model="account.account.tag">
        <field name="name">+CAPB</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.ph"/>
    </record>
    <record id="tax_tag_capital_tax_negative" model="account.account.tag">
        <field name="name">-CAPB</field>
        <field name="applicability">taxes</field>
        <field name="tax_negate">True</field>
        <field name="country_id" ref="base.ph"/>
    </record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="vat" model="account.report">
        <field name="name">2550Q</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ph"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_base" model="account.report.column">
                <field name="name">Tax base</field>
                <field name="expression_label">tax_base</field>
            </record>
            <record id="vat_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_payable" model="account.report.line">
                <field name="name">Tax Payable</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="net_vat_payable_from_56" model="account.report.line">
                        <field name="name">15 Net VAT Payable(Excess Input Tax)</field>
                        <field name="code">15</field>
                        <field name="hierarchy_level">1</field>
                        <field name="aggregation_formula">56.balance</field>
                    </record>
                    <record id="creditable_vat_withheld" model="account.report.line">
                        <field name="name">16 Creditable VAT Withheld</field>
                        <field name="code">16</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="creditable_vat_withheld_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">16B</field>
                            </record>
                        </field>
                    </record>
                    <record id="advance_vat_payments" model="account.report.line">
                        <field name="name">17 Advance VAT Payments</field>
                        <field name="code">17</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="advance_vat_payments_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">17B</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_paid_amended" model="account.report.line">
                        <field name="name">18 VAT paid in return previously filed, if this is an amended return</field>
                        <field name="code">18</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="vat_paid_amended_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">18B</field>
                            </record>
                        </field>
                    </record>
                    <record id="other" model="account.report.line">
                        <field name="name">19 Other Credits/Payment</field>
                        <field name="code">19</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="other_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">19B</field>
                            </record>
                        </field>
                    </record>
                    <record id="total_tax_cp" model="account.report.line">
                        <field name="name">20 Total Tax Credits/Payment</field>
                        <field name="code">20</field>
                        <field name="hierarchy_level">1</field>
                        <field name="aggregation_formula">16.balance + 17.balance + 18.balance + 19.balance</field>
                    </record>
                    <record id="tax_still_payable" model="account.report.line">
                        <field name="name">21 Tax Still Payable/(Excess Credits)</field>
                        <field name="code">21</field>
                        <field name="hierarchy_level">1</field>
                        <field name="aggregation_formula">15.balance - 20.balance</field>
                    </record>
                    <record id="penalties" model="account.report.line">
                        <field name="name">Add: Penalties</field>
                        <field name="hierarchy_level">1</field>
                        <field name="children_ids">
                            <record id="surcharge" model="account.report.line">
                                <field name="name">22 Surcharge</field>
                                <field name="code">22</field>
                                <field name="hierarchy_level">2</field>
                                <field name="expression_ids">
                                    <record id="surcharge_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">22B</field>
                                    </record>
                                </field>
                            </record>
                            <record id="interest" model="account.report.line">
                                <field name="name">23 Interest</field>
                                <field name="code">23</field>
                                <field name="hierarchy_level">2</field>
                                <field name="expression_ids">
                                    <record id="interest_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">23B</field>
                                    </record>
                                </field>
                            </record>
                            <record id="compromise" model="account.report.line">
                                <field name="name">24 Compromise</field>
                                <field name="code">24</field>
                                <field name="hierarchy_level">2</field>
                                <field name="expression_ids">
                                    <record id="compromise_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">24B</field>
                                    </record>
                                </field>
                            </record>
                            <record id="total_penalties" model="account.report.line">
                                <field name="name">25 Total Penalties</field>
                                <field name="code">25</field>
                                <field name="hierarchy_level">2</field>
                                <field name="hierarchy_level">1</field>
                                <field name="aggregation_formula">22.balance + 23.balance + 24.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="total_tax_payable" model="account.report.line">
                        <field name="name">26 Total Amount Payable</field>
                        <field name="code">26</field>
                        <field name="hierarchy_level">1</field>
                        <field name="aggregation_formula">21.balance + 25.balance</field>
                    </record>
                </field>
            </record>
            <record id="vat_computation_detail" model="account.report.line">
                <field name="name">Details of VAT Computation</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="sales_to_private" model="account.report.line">
                        <field name="name">31 VATable-sales to Private individuals/Corporations</field>
                        <field name="code">31</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="sales_to_private_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">31A</field>
                            </record>
                            <record id="sales_to_private_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">31B</field>
                            </record>
                        </field>
                    </record>
                    <record id="sales_to_government" model="account.report.line">
                        <field name="name">32 VATable-sales to Government</field>
                        <field name="code">32</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="sales_to_government_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">32A</field>
                            </record>
                            <record id="sales_to_government_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">32B</field>
                            </record>
                        </field>
                    </record>
                    <record id="zero_rated" model="account.report.line">
                        <field name="name">33 Zero-Rated Sales</field>
                        <field name="code">33</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="zero_rated_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">33A</field>
                            </record>
                        </field>
                    </record>
                    <record id="exempt" model="account.report.line">
                        <field name="name">34 Exempt Sales</field>
                        <field name="code">34</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="exempt_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">34A</field>
                            </record>
                        </field>
                    </record>
                    <record id="total_sales_tax" model="account.report.line">
                        <field name="name">35 Total Sales &amp; Output Tax Due</field>
                        <field name="code">35</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="total_sales_tax_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">31.tax_base + 32.tax_base + 33.tax_base + 34.tax_base</field>
                            </record>
                            <record id="total_sales_tax_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">31.balance + 32.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="input_tax_carried_over" model="account.report.line">  <!-- todo this could be automated if we know where from -->
                        <field name="name">36 Input Tax Carried Over from Previous Quarter</field>
                        <field name="code">36</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="input_tax_carried_over_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">36B</field>
                            </record>
                        </field>
                    </record>
                    <record id="input_tax_deferred" model="account.report.line">
                        <field name="name">37 Input Tax Deferred on Capital Goods Exceeding P1 Million from Previous Quarter</field>
                        <field name="code">37</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="input_tax_deferred_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">37B</field>
                            </record>
                        </field>
                    </record>
                    <record id="transitional_input_tax" model="account.report.line">
                        <field name="name">38 Transitional Input Tax</field>
                        <field name="code">38</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="transitional_input_tax_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">38B</field>
                            </record>
                        </field>
                    </record>
                    <record id="presumptive_input_tax" model="account.report.line">
                        <field name="name">39 Presumptive Input Tax</field>
                        <field name="code">39</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="presumptive_input_tax_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">39B</field>
                            </record>
                        </field>
                    </record>
                    <record id="others_of_allowable_input_tax" model="account.report.line">
                        <field name="name">40 Others</field>
                        <field name="code">40</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="others_of_allowable_input_tax_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">40B</field>
                            </record>
                        </field>
                    </record>
                    <record id="total_of_allowable_input_tax" model="account.report.line">
                        <field name="name">41 Total</field>
                        <field name="code">41</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="total_of_allowable_input_tax_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">36.balance + 37.balance + 38.balance + 39.balance + 40.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="domestic_purchase" model="account.report.line">
                        <field name="name">42 Domestic Purchases</field>
                        <field name="code">42</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <!-- 42 -->
                            <record id="domestic_purchase_formula_vat_base" model="account.report.expression">
                                <field name="label">_vat_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">42A</field>
                            </record>
                            <record id="domestic_purchase_formula_vat_balance" model="account.report.expression">
                                <field name="label">_vat_balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">42B</field>
                            </record>
                            <!-- 42 service -->
                            <record id="domestic_purchase_formula_service_base" model="account.report.expression">
                                <field name="label">_service_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">42SA</field>
                            </record>
                            <record id="domestic_purchase_formula_service_balance" model="account.report.expression">
                                <field name="label">_service_balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">42SB</field>
                            </record>
                            <!-- 42 capital -->
                            <record id="domestic_purchase_formula_capital_base" model="account.report.expression">
                                <field name="label">_capital_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CAPA</field>
                            </record>
                            <record id="domestic_purchase_formula_capital_balance" model="account.report.expression">
                                <field name="label">_capital_balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">CAPB</field>
                            </record>
                            <!-- Aggregate -->
                            <record id="domestic_purchase_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">42._vat_base + 42._service_base + 42._capital_base</field>
                            </record>
                            <record id="domestic_purchase_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">42._vat_balance + 42._service_balance + 42._capital_balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="service_by_non_residents" model="account.report.line">
                        <field name="name">43 Services Rendered by Non-residents</field>
                        <field name="code">43</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="service_by_non_residents_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">43A</field>
                            </record>
                            <record id="service_by_non_residents_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">43B</field>
                            </record>
                        </field>
                    </record>
                    <record id="importations" model="account.report.line">
                        <field name="name">44 Importations</field>
                        <field name="code">44</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="importations_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">44A</field>
                            </record>
                            <record id="importations_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">44B</field>
                            </record>
                        </field>
                    </record>
                    <record id="others_current_transaction" model="account.report.line">
                        <field name="name">45 Others</field>
                        <field name="code">45</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="others_current_transaction_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">45A</field>
                            </record>
                            <record id="others_current_transaction_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">45B</field>
                            </record>
                        </field>
                    </record>
                    <record id="domestic_purchase_untaxed" model="account.report.line">
                        <field name="name">46 Domestic Purchase with No Input Tax</field>
                        <field name="code">46</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <!-- 42 service -->
                            <record id="domestic_purchase_untaxed_formula_ex_base" model="account.report.expression">
                                <field name="label">_exempt_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">46E</field>
                            </record>
                            <record id="domestic_purchase_untaxed_formula_zr_base" model="account.report.expression">
                                <field name="label">_zr_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">46ZR</field>
                            </record>
                            <!-- Aggregate -->
                            <record id="domestic_purchase_untaxed_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">46._exempt_base + 46._zr_base</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_exempt_importations" model="account.report.line">
                        <field name="name">47 VAT-Exempt Importations</field>
                        <field name="code">47</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="vat_exempt_importations_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">47A</field>
                            </record>
                        </field>
                    </record>
                    <record id="total_current_tax" model="account.report.line">
                        <field name="name">48 Total Current Purchase/Input Tax</field>
                        <field name="code">48</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="total_current_tax_base" model="account.report.expression">
                                <field name="label">tax_base</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">42.tax_base + 43.tax_base + 44.tax_base + 45.tax_base + 46.tax_base + 47.tax_base</field>
                            </record>
                            <record id="total_current_tax_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">42.balance + 43.balance + 44.balance + 45.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="total_available_input_tax" model="account.report.line">
                        <field name="name">49 Total Available Input Tax</field>
                        <field name="code">49</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="total_available_input_tax_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">41.balance + 48.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="input_tax_exceeding_1_m" model="account.report.line">
                        <field name="name">50 Input Tax on Purchases/Importation of Capital Goods exceeding P1 Million deferred for the succeeding period</field>
                        <field name="code">50</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="input_tax_exceeding_1_m_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">50B</field>
                            </record>
                        </field>
                    </record>
                    <record id="input_tax_vat_exempt" model="account.report.line">
                        <field name="name">51 Input Tax Attributable to VAT Exempt Sales</field>
                        <field name="code">51</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="input_tax_vat_exempt_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">51B</field>
                            </record>
                        </field>
                    </record>
                    <record id="vat_refund_ttc_claimed" model="account.report.line">
                        <field name="name">52 VAT refund/TCC claimed</field>
                        <field name="code">52</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="vat_refund_ttc_claimed_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">52B</field>
                            </record>
                        </field>
                    </record>
                    <record id="others_net_vay_payable" model="account.report.line">
                        <field name="name">53 Others</field>
                        <field name="code">53</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="others_net_vay_payable_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">53B</field>
                            </record>
                        </field>
                    </record>
                    <record id="total_deductions_from_input_tax" model="account.report.line">
                        <field name="name">54 Total Deductions from Input Tax</field>
                        <field name="code">54</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="total_deductions_from_input_tax_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">50.balance + 51.balance + 52.balance + 53.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="total_allowable_input_tax" model="account.report.line">
                        <field name="name">55 Total Allowable Input Tax</field>
                        <field name="code">55</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="total_allowable_input_tax_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">49.balance - 54.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="net_vat_payable" model="account.report.line">
                        <field name="name">56 Net VAT Payable/(Excess Input Tax)</field>
                        <field name="code">56</field>
                        <field name="hierarchy_level">1</field>
                        <field name="expression_ids">
                            <record id="net_vat_payable_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">35.balance - 55.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-ph.csv

```csv
"id","name","code","account_type","reconcile"
"l10n_ph_100000","Bank Suspense Account","100000","asset_current","False"
"l10n_ph_110000","Accounts Receivable","110000","asset_receivable","True"
"l10n_ph_110003","Accounts Receivable (POS)","110003","asset_receivable","True"
"l10n_ph_110100","Advances to Suppliers","110100","asset_prepayments","False"
"l10n_ph_110101","Deposits","110101","asset_prepayments","False"
"l10n_ph_110102","Employees Receivable-Cash Advance","110102","asset_current","False"
"l10n_ph_110103","Employees Receivable-Loan","110103","asset_current","False"
"l10n_ph_110104","ER - Others","110104","asset_current","False"
"l10n_ph_110105","Prepaid Insurance","110105","asset_prepayments","False"
"l10n_ph_110106","Prepaid Pension - contribution","110106","asset_prepayments","False"
"l10n_ph_110200","Prepaid Tax - Tax Credit Certificate","110200","asset_current","False"
"l10n_ph_110201","Input VAT 12%","110201","asset_current","False"
"l10n_ph_110202","Deferred Input VAT 12%","110202","asset_current","False"
"l10n_ph_110203","Deferred Tax Asset","110203","asset_current","False"
"l10n_ph_110205","Creditable Income Tax","110205","asset_current","False"
"l10n_ph_110206","Tax Withheld by Customer","110206","asset_current","False"
"l10n_ph_110207","Deferred Tax Withheld by Customer","110207","asset_current","False"
"l10n_ph_110300","Inventory","110300","asset_current","False"
"l10n_ph_110301","Purchases","110301","asset_current","False"
"l10n_ph_110302","Stock Interim (Received)","110302","asset_current","True"
"l10n_ph_110303","Stock Interim (Delivered)","110303","asset_current","True"
"l10n_ph_110304","Stock Interim (Production)","110304","asset_current","True"
"l10n_ph_110400","Other Non-current Asset","110400","asset_non_current","False"
"l10n_ph_110500","Building and Leasehold Improvements","110500","asset_fixed","False"
"l10n_ph_110501","Furniture, Fixtures and Equipment","110501","asset_fixed","False"
"l10n_ph_110502","Vehicles","110502","asset_fixed","False"
"l10n_ph_110503","Computers","110503","asset_fixed","False"
"l10n_ph_110504","Accum Depreciation-Bldg & Leasehold Imp","110504","asset_fixed","False"
"l10n_ph_110505","Accum Depreciation-Furniture, Fixtures and Equipment","110505","asset_fixed","False"
"l10n_ph_110506","Accum Depreciation-Vehicles","110506","asset_fixed","False"
"l10n_ph_110507","Accum Depreciation-Computers","110507","asset_fixed","False"
"l10n_ph_200000","Accounts Payable","200000","liability_payable","True"
"l10n_ph_200200","Dividends Payable - Common Shares","200200","liability_current","False"
"l10n_ph_200201","Unearned Revenue","200201","liability_current","False"
"l10n_ph_200202","Customer Deposits Received","200202","liability_current","False"
"l10n_ph_200300","Output VAT 12% - Actual","200300","liability_current","False"
"l10n_ph_200301","Deferred Output VAT 12% - Actual","200301","liability_current","False"
"l10n_ph_200302","Tax Payable-Withholding Tax Compensation","200302","liability_current","False"
"l10n_ph_200303","Tax Pay-Withholding Tax Source","200303","liability_current","False"
"l10n_ph_200304","Deferred Tax Pay-Withholding Tax Source","200304","liability_current","False"
"l10n_ph_200305","Taxes Payable-Final Withholding Tax","200305","liability_current","False"
"l10n_ph_200306","Tax Payable-Withholding Tax at Source (Accrual)","200306","liability_current","False"
"l10n_ph_200307","Income Tax Payable","200307","liability_current","False"
"l10n_ph_200308","Accrued Fringe Benefit Tax","200308","liability_current","False"
"l10n_ph_200400","Other Non-current Liability","200400","liability_non_current","False"
"l10n_ph_300000","Capital Stock","300000","equity","False"
"l10n_ph_300001","Additional Paid in Capital","300001","equity","False"
"l10n_ph_300002","Retained Earnings","300002","equity","False"
"l10n_ph_430400","Sales/Revenues","430400","income","False"
"l10n_ph_510000","Cost of Goods Sold","510000","expense_direct_cost","False"
"l10n_ph_511100","COGS - Purchases","511100","expense_direct_cost","False"
"l10n_ph_511600","COGS - Freight Costs","511600","expense_direct_cost","False"
"l10n_ph_511700","COGS - Direct Labor","511700","expense_direct_cost","False"
"l10n_ph_511800","COGS - Factory/Processing Overhead","511800","expense_direct_cost","False"
"l10n_ph_620000","Admin Expense","620000","expense","False"
"l10n_ph_620001","Marketing Expense","620001","expense","False"
"l10n_ph_620100","Lease-Corporate Offices","620100","expense","False"
"l10n_ph_621000","Insurance Expenses","621000","expense","False"
"l10n_ph_622000","Repairs & Maintenance","622000","expense","False"
"l10n_ph_623000","Salaries & Wages","623000","expense","False"
"l10n_ph_623001","Overtime Pay","623001","expense","False"
"l10n_ph_623002","SSS Contribution","623002","expense","False"
"l10n_ph_623003","HDMF Contribution","623003","expense","False"
"l10n_ph_623004","PHIC Contribution","623004","expense","False"
"l10n_ph_623005","Meal Allowance","623005","expense","False"
"l10n_ph_623006","Other Allowance","623006","expense","False"
"l10n_ph_623007","13th Month Pay","623007","expense","False"
"l10n_ph_623008","Retirement Expenses","623008","expense","False"
"l10n_ph_623009","Training and Webinar","623009","expense","False"
"l10n_ph_623014","Sick Leave Expenses","623014","expense","False"
"l10n_ph_623015","Vacation Leave Expenses","623015","expense","False"
"l10n_ph_624000","Taxes and Licenses","624000","expense","False"
"l10n_ph_625000","Utilities Expenses","625000","expense","False"
"l10n_ph_626000","Security Services","626000","expense","False"
"l10n_ph_626001","Professional Fees","626001","expense","False"
"l10n_ph_627000","Travel and Transportation","627000","expense","False"
"l10n_ph_628000","Advertising and Promotion","628000","expense","False"
"l10n_ph_629000","Depreciation Expense","629000","expense_depreciation","False"
"l10n_ph_632000","Commission Expense","632000","expense","False"
"l10n_ph_633000","Supplies Expenses","633000","expense","False"
"l10n_ph_634000","Miscellaneous Expenses","634000","expense","False"
"l10n_ph_634001","Bank Fees","634001","expense","False"
"l10n_ph_635000","Prov DA-Billed Accts","635000","expense","False"
"l10n_ph_635001","Prov-Probable Expenses","635001","expense","False"
"l10n_ph_635002","Prov for Inc Tax - Creditable","635002","expense","False"
"l10n_ph_635003","Provision for Tax - Final","635003","expense","False"
"l10n_ph_710100","Forex Gain","710100","income_other","False"
"l10n_ph_710101","Forex Loss","710101","expense","False"
"l10n_ph_710102","Cash Difference Gain","710102","income_other","False"
"l10n_ph_710103","Cash Difference Loss","710103","expense","False"
"l10n_ph_710200","Gain/Loss- Sale FA","710200","expense","False"
"l10n_ph_710201","Loss on FA Disposal","710201","expense","False"
"l10n_ph_710300","Interest Inc-Savings","710300","income_other","False"
"l10n_ph_710301","Interest Income-Savings Local","710301","income_other","False"
"l10n_ph_710400","Other Income","710400","income_other","False"
"l10n_ph_710500","Interest Expense-Suppliers' Credit Local","710500","income_other","False"
"l10n_ph_999999","Undistributed Profits/Losses","999999","equity_unaffected","False"

```

## File: data\template\account.fiscal.position-ph.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","tax_ids/tax_src_id","tax_ids/tax_dest_id"
"l10n_ph_fiscal_position_vat_registered","1","VAT Registered","1","1","base.ph","l10n_ph_tax_sale_vat_exempt","l10n_ph_tax_sale_vat_12"
"","","","","","","l10n_ph_tax_purchase_vat_exempt","l10n_ph_tax_purchase_vat_12"
"l10n_ph_fiscal_position_vat_exempt","2","VAT Exempt","1","","base.ph","l10n_ph_tax_sale_vat_12","l10n_ph_tax_sale_vat_exempt"
"","","","","","","l10n_ph_tax_purchase_vat_12","l10n_ph_tax_purchase_vat_exempt"

```

## File: data\template\account.tax-ph.csv

```csv
"id","sequence","name","description","active","invoice_label","type_tax_use","amount","amount_type","tax_group_id","tax_scope","l10n_ph_atc","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids"
"l10n_ph_tax_sale_vat_12","10","12%","","True","12% VAT","sale","12.0","percent",l10n_ph_tax_group_vat_12,"","",100,"base","invoice","","+31A"
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200300","+31B"
"","","","","","","","","","","","","100","base","refund","","-31A"
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200300","-31B"
"l10n_ph_tax_sale_vat_exempt","10","0% EXEMPT","","True","VAT Exempt","sale","0.0","percent","l10n_ph_tax_group_vat_exempt","","","100","base","invoice","","+34A"
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200300",""
"","","","","","","","","","","","","100","base","refund","","-34A"
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200300",""
"l10n_ph_tax_sale_vat_zero_rated","10","0% ZR","","True","Zero Rated","sale","0.0","percent","l10n_ph_tax_group_vat_exempt","","","100","base","invoice","","+33A"
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200300",""
"","","","","","","","","","","","","100","base","refund","","-33A"
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200300",""
"l10n_ph_tax_sale_vat_12_gov","10","12% GOV","","False","12% GOV","sale","12.0","percent","l10n_ph_tax_group_vat_12","","","100","base","invoice","","+32A"
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200300","+32B"
"","","","","","","","","","","","","100","base","refund","","-32A"
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200300","-32B"
"l10n_ph_tax_purchase_vat_12","20","12%","","True","12% VAT","purchase","12.0","percent","l10n_ph_tax_group_vat_12","","","100","base","invoice","","+42A"
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_110201","+42B"
"","","","","","","","","","","","","100","base","refund","","-42A"
"","","","","","","","","","","","","100","tax","refund","l10n_ph_110201","-42B"
"l10n_ph_tax_purchase_vat_12_service","20","12% S","","True","12% Services","purchase","12.0","percent","l10n_ph_tax_group_vat_12","service","","100","base","invoice","","+42SA"
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_110201","+42SB"
"","","","","","","","","","","","","100","base","refund","","-42SA"
"","","","","","","","","","","","","100","tax","refund","l10n_ph_110201","-42SB"
"l10n_ph_tax_purchase_vat_exempt","20","0% EXEMPT","","True","VAT Exempt","purchase","0.0","percent","l10n_ph_tax_group_vat_exempt","","","100","base","invoice","","+46E"
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_110201",""
"","","","","","","","","","","","","100","base","refund","","-46E"
"","","","","","","","","","","","","100","tax","refund","l10n_ph_110201",""
"l10n_ph_tax_purchase_vat_exempt_import","20","0% EXEMPT I","","False","VAT Exempt Imports","purchase","0.0","percent","l10n_ph_tax_group_vat_exempt","","","100","base","invoice","","+47A"
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_110201",""
"","","","","","","","","","","","","100","base","refund","","-47A"
"","","","","","","","","","","","","100","tax","refund","l10n_ph_110201",""
"l10n_ph_tax_purchase_vat_zero_rated","20","0% ZR","","True","Zero Rated","purchase","0.0","percent","l10n_ph_tax_group_vat_exempt","","","100","base","invoice","","+46ZR"
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_110201",""
"","","","","","","","","","","","","100","base","refund","","-46ZR"
"","","","","","","","","","","","","100","tax","refund","l10n_ph_110201",""
"l10n_ph_tax_purchase_vat_12_service_non_residents","20","12% S NR","","False","12% VAT Non Resident","purchase","12.0","percent","l10n_ph_tax_group_vat_12","service","","100","base","invoice","","+43A"
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_110201","+43B"
"","","","","","","","","","","","","100","base","refund","","-43A"
"","","","","","","","","","","","","100","tax","refund","l10n_ph_110201","-43B"
"l10n_ph_tax_purchase_vat_12_capital","20","12% C","","True","12% Capital","purchase","12.0","percent","l10n_ph_tax_group_capital_12","consu","","100","base","invoice","","+CAPA"
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_110201","+CAPB"
"","","","","","","","","","","","","100","base","refund","","-CAPA"
"","","","","","","","","","","","","100","tax","refund","l10n_ph_110201","-CAPB"
"l10n_ph_tax_purchase_vat_12_import","20","12% I","","False","12% Imports","purchase","12.0","percent","l10n_ph_tax_group_capital_12","consu","","100","base","invoice","","+44A"
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_110201","+44B"
"","","","","","","","","","","","","100","base","refund","","-44A"
"","","","","","","","","","","","","100","tax","refund","l10n_ph_110201","-44B"
"l10n_ph_tax_purchase_wi010","30","5% P F","5% WI010 - Prof Fees","True","Prof Fees","purchase","-5.0","percent","l10n_ph_tax_group_wht_5","","WI010","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wi011","30","10% WI P F","10% WI011 - Prof Fees","True","Prof Fees","purchase","-10.0","percent","l10n_ph_tax_group_wht_10","","WI011","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wi100","40","5% WI G R","5% WI100 - Gross rental of property","True","Gross rental of property","purchase","-5.0","percent","l10n_ph_tax_group_wht_5","","WI100","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wi120","50","2% WI Cont","2% WI120 - Contractors","True","Contractors","purchase","-2.0","percent","l10n_ph_tax_group_wht_2","","WI120","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wi139","60","5% Com S","5% WI139 - Commission of service fees","True","Commission of service fees","purchase","-5.0","percent","l10n_ph_tax_group_wht_5","","WI139","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wi140","60","10% Com S","10% WI139 - Commission of service fees","True","Commission of service fees","purchase","-10.0","percent","l10n_ph_tax_group_wht_10","","WI140","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wi158_p5","70","0.5% WI C C","0.5% WI158 - Credit card companies","True","Credit card companies","purchase","-0.5","percent","l10n_ph_tax_group_wht_p5","","WI158","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wi640","80","1% WI640 Sup G","1% WI640 - Supplier of goods","True","Supplier of goods","purchase","-1.0","percent","l10n_ph_tax_group_wht_1","","WI640","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wi157","80","2% WI Sup S","2% WI157 - Supplier of services","True","Supplier of services","purchase","-2.0","percent","l10n_ph_tax_group_wht_2","","WI157","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wi158_1","90","1% WI158 Sup G","1% WI158 - Supplier of goods by top w/holding agents","True","Supplier of goods by top w/holding agents","purchase","-1.0","percent","l10n_ph_tax_group_wht_1","","WI158","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wi160","90","2% WI Sup G","2% WI160 - Supplier of goods by top w/holding agents","True","Supplier of goods by top w/holding agents","purchase","-2.0","percent","l10n_ph_tax_group_wht_2","","WI160","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wi515","100","5% WI C","5% WI515 - Commission, rebates, discounts","True","Commission, rebates, discounts","purchase","-5.0","percent","l10n_ph_tax_group_wht_5","","WI515","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wi516","100","10% WI C","10% WI516 - Commission, rebates, discounts","True","Commission, rebates, discounts","purchase","-10.0","percent","l10n_ph_tax_group_wht_10","","WI516","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wc010","110","10% WC P F","10% WC010 - Prof Fees","True","Prof Fees","purchase","-10.0","percent","l10n_ph_tax_group_wht_10","","WC010","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wc011","110","15% P F","15% WC011 - Prof Fees","True","Prof Fees","purchase","-15.0","percent","l10n_ph_tax_group_wht_15","","WC011","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wc100","120","5% WC G R","5% WC100 - Gross rental of property","True","Gross rental of property","purchase","-5.0","percent","l10n_ph_tax_group_wht_5","","WC100","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wc120","130","2% WC Cont","2% WC120 - Contractors","True","Contractors","purchase","-2.0","percent","l10n_ph_tax_group_wht_2","","WC120","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wc139","140","10% WC139 C","10% WC139 - Commission of service fees","True","Commission of service fees","purchase","-10.0","percent","l10n_ph_tax_group_wht_10","","WC139","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wc140","140","15% C","15% WC140 - Commission of service fees","True","Commission of service fees","purchase","-15.0","percent","l10n_ph_tax_group_wht_15","","WC140","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wc158_p5","150","0.5% WC C C","0.5% WC158 - Credit card companies","True","Credit card companies","purchase","-0.5","percent","l10n_ph_tax_group_wht_p5","","WC158","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wc640","160","1% WC640 Sup G","1% WC640 - Supplier of goods","True","Supplier of goods","purchase","-1.0","percent","l10n_ph_tax_group_wht_1","","WC640","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wc157","160","2% WC Sup S","2% WC157 - Supplier of services","True","Supplier of services","purchase","-2.0","percent","l10n_ph_tax_group_wht_2","","WC157","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wc158_1","170","1% WC158 Sup G","1% WC158 - Supplier of goods by top w/holding agents","True","Supplier of goods by top w/holding agents","purchase","-1.0","percent","l10n_ph_tax_group_wht_1","","WC158","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wc160","170","2% WC Sup G","2% WC160 - Supplier of goods by top w/holding agents","True","Supplier of goods by top w/holding agents","purchase","-2.0","percent","l10n_ph_tax_group_wht_2","","WC160","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wc515","180","5% WC C","5% WC515 - Commission, rebates, discounts","True","Commission, rebates, discounts","purchase","-5.0","percent","l10n_ph_tax_group_wht_5","","WC515","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""
"l10n_ph_tax_purchase_wc516","180","10% WC516 C","10% WC516 - Commission, rebates, discounts","True","Commission, rebates, discounts","purchase","-10.0","percent","l10n_ph_tax_group_wht_10","","WC516","100","base","invoice","",""
"","","","","","","","","","","","","100","tax","invoice","l10n_ph_200303",""
"","","","","","","","","","","","","100","base","refund","",""
"","","","","","","","","","","","","100","tax","refund","l10n_ph_200303",""

```

## File: data\template\account.tax.group-ph.csv

```csv
"id","name"
"l10n_ph_tax_group_vat_12","VAT 12%"
"l10n_ph_tax_group_vat_exempt","VAT Exempt"
"l10n_ph_tax_group_capital_12","Capital 12%"
"l10n_ph_tax_group_wht_p5","WHT 0.5%"
"l10n_ph_tax_group_wht_1","WHT 1%"
"l10n_ph_tax_group_wht_2","WHT 2%"
"l10n_ph_tax_group_wht_5","WHT 5%"
"l10n_ph_tax_group_wht_10","WHT 10%"
"l10n_ph_tax_group_wht_15","WHT 15%"

```

## File: migrations\1.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'ph')], order="parent_path"):
        env['account.chart.template'].try_loading('ph', company)

```

## File: models\account_move.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from odoo.exceptions import UserError


class AccountMove(models.Model):
    _inherit = "account.move"

    def action_open_l10n_ph_2307_wizard(self):
        vendor_bills = self.filtered_domain([('move_type', '=', 'in_invoice')])
        if vendor_bills:
            wizard_action = self.env["ir.actions.act_window"]._for_xml_id("l10n_ph.view_l10n_ph_2307_wizard_act_window")
            wizard_action.update({
                'context': {'default_moves_to_export': vendor_bills.ids}
            })
            return wizard_action
        else:
            raise UserError(_('Only Vendor Bills are available.'))

```

## File: models\account_payment.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import _, models
from odoo.exceptions import UserError


class AccountPayment(models.Model):
    _inherit = "account.payment"

    def action_open_l10n_ph_2307_wizard(self):
        self.ensure_one()
        if self.payment_type == 'outbound':
            wizard_action = self.env["ir.actions.act_window"]._for_xml_id("l10n_ph.view_l10n_ph_2307_wizard_act_window")
            wizard_action.update({
                'context': {'default_moves_to_export': self.reconciled_bill_ids.ids}
            })
            return wizard_action
        else:
            raise UserError(_('Only Outbound Payment is available.'))

```

## File: models\account_tax.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountTax(models.Model):
    _inherit = "account.tax"

    l10n_ph_atc = fields.Char("Philippines ATC")

```

## File: models\res_partner.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, api, models


class ResPartner(models.Model):
    _inherit = "res.partner"

    branch_code = fields.Char("Branch Code", default='000', compute='_compute_branch_code', store=True)
    first_name = fields.Char("First Name")
    middle_name = fields.Char("Middle Name")
    last_name = fields.Char("Last Name")

    @api.model
    def _commercial_fields(self):
        return super()._commercial_fields() + ['branch_code']

    @api.depends('vat', 'country_id')
    def _compute_branch_code(self):
        for partner in self:
            branch_code = '000'
            if partner.country_id.code == 'PH' and partner.vat:
                match = partner.__check_vat_ph_re.match(partner.vat)
                branch_code = match and match.group(1) and match.group(1)[1:] or branch_code
            partner.branch_code = branch_code

```

## File: models\template_ph.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('ph')
    def _get_ph_template_data(self):
        return {
            'property_account_receivable_id': 'l10n_ph_110000',
            'property_account_payable_id': 'l10n_ph_200000',
            'property_account_income_categ_id': 'l10n_ph_430400',
            'property_account_expense_categ_id': 'l10n_ph_620000',
            'property_stock_valuation_account_id': 'l10n_ph_110300',
            'property_stock_account_input_categ_id': 'l10n_ph_110302',
            'property_stock_account_output_categ_id': 'l10n_ph_110303',
            'code_digits': '6',
        }

    @template('ph', 'res.company')
    def _get_ph_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.ph',
                'bank_account_code_prefix': '1000',
                'cash_account_code_prefix': '1001',
                'transfer_account_code_prefix': '1002',
                'account_default_pos_receivable_account_id': 'l10n_ph_110003',
                'income_currency_exchange_account_id': 'l10n_ph_710100',
                'expense_currency_exchange_account_id': 'l10n_ph_710101',
                'account_journal_suspense_account_id': 'l10n_ph_100000',
                'default_cash_difference_income_account_id': 'l10n_ph_710102',
                'default_cash_difference_expense_account_id': 'l10n_ph_710103',
                'account_sale_tax_id': 'l10n_ph_tax_sale_vat_12',
                'account_purchase_tax_id': 'l10n_ph_tax_purchase_vat_12',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_ph
from . import res_partner
from . import account_move
from . import account_payment
from . import account_tax

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
generate_2307_wizard,access_generate_2307_wizard,model_l10n_ph_2307_wizard,account.group_account_invoice,1,1,1,1

```

## File: views\account_move_views.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <record id="action_account_move_bir_2307" model="ir.actions.server">
        <field name="name">Download BIR 2307 XLS</field>
        <field name="groups_id" eval="[(4, ref('account.group_account_invoice'))]"/>
        <field name="model_id" ref="account.model_account_move"/>
        <field name="binding_model_id" ref="account.model_account_move"/>
        <field name="binding_view_types">list,form</field>
        <field name="state">code</field>
        <field name="code">action = records.action_open_l10n_ph_2307_wizard()</field>
    </record>
</odoo>

```

## File: views\account_payment_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="action_account_payment_bir_2307" model="ir.actions.server">
        <field name="name">Download BIR 2307 XLS</field>
        <field name="groups_id" eval="[(4, ref('account.group_account_invoice'))]"/>
        <field name="model_id" ref="account.model_account_payment"/>
        <field name="binding_model_id" ref="account.model_account_payment"/>
        <field name="binding_view_types">form</field>
        <field name="state">code</field>
        <field name="code">action = record.action_open_l10n_ph_2307_wizard()</field>
    </record>
</odoo>

```

## File: views\account_tax_views.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo>
    <record id="view_tax_form" model="ir.ui.view">
        <field name="name">account.tax.view.form.inherit.l10n_ph</field>
        <field name="model">account.tax</field>
        <field name="inherit_id" ref="account.view_tax_form" />
        <field name="arch" type="xml">
            <notebook position="inside">
                <page name="l10n_ph" string="Philippines" invisible="country_code != 'PH'">
                    <group>
                        <group>
                            <field name="l10n_ph_atc" />
                        </group>
                    </group>
                </page>
            </notebook>
        </field>
    </record>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<odoo>
    <record id="view_partner_form" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.l10n_ph_bir</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="account.view_partner_property_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="after">
                <field name="branch_code" invisible="'PH' not in fiscal_country_codes"/>
                <field name="first_name" invisible="'PH' not in fiscal_country_codes or is_company"/>
                <field name="middle_name" invisible="'PH' not in fiscal_country_codes or is_company"/>
                <field name="last_name" invisible="'PH' not in fiscal_country_codes or is_company"/>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: wizard\generate_2307_wizard.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64
import io
import re
import xlwt

from odoo import fields, models
from odoo.tools.misc import format_date


COLUMN_HEADER_MAP = {
    "Reporting_Month": "invoice_date",
    "Vendor_TIN": "vat",
    "branchCode": "branch_code",
    "companyName": "company_name",
    "surName": "last_name",
    "firstName": "first_name",
    "middleName": "middle_name",
    "address": "address",
    "nature": "product_name",
    "ATC": "atc",
    "income_payment": "price_subtotal",
    "ewt_rate": "amount",
    "tax_amount": "tax_amount",
}

class Generate2307Wizard(models.TransientModel):
    _name = "l10n_ph_2307.wizard"
    _description = "Exports 2307 data to a XLS file."

    moves_to_export = fields.Many2many("account.move", string="Joural To Include")
    generate_xls_file = fields.Binary(
        "Generated file",
        help="Technical field used to temporarily hold the generated XLS file before its downloaded."
    )

    def _write_single_row(self, worksheet, worksheet_row, values):
        for index, field in enumerate(COLUMN_HEADER_MAP.values()):
            worksheet.write(worksheet_row, index, label=values[field])

    def _write_rows(self, worksheet, moves):
        worksheet_row = 0
        for move in moves:
            worksheet_row += 1
            partner = move.partner_id
            partner_address_info = [partner.street, partner.street2, partner.city, partner.state_id.name, partner.country_id.name]
            values = {
                'invoice_date': format_date(self.env, move.invoice_date, date_format="MM/dd/yyyy"),
                'vat': re.sub(r'\-', '', partner.vat)[:9] if partner.vat else '',
                'branch_code': partner.branch_code or '000',
                'company_name': partner.commercial_partner_id.name,
                'first_name': partner.first_name or '',
                'middle_name': partner.middle_name or '',
                'last_name': partner.last_name or '',
                'address': ', '.join([val for val in partner_address_info if val])
            }
            aggregated_taxes = move._prepare_invoice_aggregated_taxes()
            for invoice_line, tax_details_for_line in aggregated_taxes['tax_details_per_record'].items():
                for tax_detail in tax_details_for_line['tax_details'].values():
                    tax = tax_detail['tax']
                    if not tax.l10n_ph_atc:
                        continue

                    product_name = invoice_line.product_id.name or invoice_line.name
                    values['product_name'] = re.sub(r'[\(\)]', '', product_name) if product_name else ""
                    values['atc'] = tax.l10n_ph_atc
                    values['price_subtotal'] = tax_detail['base_amount']
                    values['amount'] = tax.amount
                    values['tax_amount'] = tax_detail['tax_amount']
                    self._write_single_row(worksheet, worksheet_row, values)
                    worksheet_row += 1

    def action_generate(self):
        """ Generate a xls format file for importing to
        https://bir-excel-uploader.com/excel-file-to-bir-dat-format/#bir-form-2307-settings.
        This website will then generate a BIR 2307 format excel file for uploading to the
        PH government.
        """
        self.ensure_one()

        file_data = io.BytesIO()
        workbook = xlwt.Workbook(encoding='utf-8')
        worksheet = workbook.add_sheet('Form2307')

        for index, col_header in enumerate(COLUMN_HEADER_MAP.keys()):
            worksheet.write(0, index, label=col_header)

        self._write_rows(worksheet, self.moves_to_export)

        workbook.save(file_data)
        file_data.seek(0)
        self.generate_xls_file = base64.b64encode(file_data.read())

        return {
            "type": "ir.actions.act_url",
            "target": "self",
            "url": "/web/content?model=l10n_ph_2307.wizard&download=true&field=generate_xls_file&filename=Form_2307.xls&id={}".format(self.id),
        }

```

## File: wizard\generate_2307_wizard_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_ph_2307_wizard_view_form" model="ir.ui.view">
        <field name="name">l10n_ph_2307.wizard.form</field>
        <field name="model">l10n_ph_2307.wizard</field>
        <field name="arch" type="xml">
            <form string="Generate BIR 2307 Report">
                This will export a XLS file for BIR 2307.
                <group>
                    <field name="moves_to_export" nolabel="1" readonly="1" colspan="2">
                        <tree>
                            <field name="name" optional="show"/>
                            <field name="invoice_partner_display_name" string="Vendor"/>
                            <field name="invoice_date" string="Bill Date" readonly="state != 'draft'"/>
                            <field name="invoice_date_due"/>
                            <field name="currency_id" column_invisible="True" readonly="state in ['cancel', 'posted']"/>
                            <field name="amount_tax_signed" string="Tax" sum="Total" optional="hide" modifiers="{'readonly':true}" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                            <field name="amount_total_signed" string="Total" sum="Total" decoration-bf="1" optional="show" widget="monetary" options="{'currency_field': 'currency_id'}"/>
                            <field name="state" widget="badge" decoration-success="state == 'posted'" decoration-info="state == 'draft'" optional="show" on_change="1" modifiers="{'readonly':true, 'required':true}"/>
                        </tree>
                    </field>
                </group>
                <footer>
                    <button string="Generate" type="object" name="action_generate" class="btn btn-primary" data-hotkey="q"/>
                    <button string="Cancel" special="cancel" data-hotkey="x" class="btn btn-secondary"/>
                </footer>
            </form>
        </field>
    </record>
    <record id="view_l10n_ph_2307_wizard_act_window" model="ir.actions.act_window">
        <field name="name">BIR 2307 Report</field>
        <field name="res_model">l10n_ph_2307.wizard</field>
        <field name="view_mode">form</field>
        <field name="target">new</field>
    </record>
</odoo>

```

## File: wizard\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import generate_2307_wizard

```

